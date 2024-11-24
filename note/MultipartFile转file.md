# MultipartFile

1. 后端服务通过feign调用对方的的文件上传服务需要本地file转为MultipartFile

```java
public class MultipartFileUtils {

    public static MultipartFile toMultipartFile(File file) {
        FileItem fileItem = createFileItem(file);
        MultipartFile commonsMultipartFile = new CommonsMultipartFile(fileItem);
        return commonsMultipartFile;
    }

    @SneakyThrows
    private static FileItem createFileItem(File file) {
        FileItem fileItem = new DiskFileItem("mainFile", Files.probeContentType(file.toPath()), false, file.getName()
                , (int) file.length(), file.getParentFile());
        try {
            int bytesRead = 0;
            byte[] buffer = new byte[10*1024*1024];
            FileInputStream fis = new FileInputStream(file);
            OutputStream os = fileItem.getOutputStream();
            while ((bytesRead = fis.read(buffer, 0, 8192)) != -1) {
                os.write(buffer, 0, bytesRead);
            }
            os.close();
            fis.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return fileItem;
    }
}
```

2. feign接口

```java
   /**
     * 分片上传
     * @return
     */
    @PostMapping(value = "/file/upload/chunk",consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    GisResultVO<Boolean> uploadChunk(@RequestPart("upfile") MultipartFile file,
                                     @RequestParam("bizType") String bizType,
                                     @RequestParam("chunkNumber") Integer chunkNumber,
                                     @RequestParam("identifier") String identifier,
                                     @RequestParam("totalChunks") Integer totalChunks,
                                     @RequestParam("uploadId") String uploadId,
                                     @RequestParam("filename") String filename);

```

3. 后端分片调用

```java
  public ModelGisResourceVO addGisResource(Integer personId, ModelGisResourceAddDTO dto) {
        // 获取文件上传到gis平台
        FileVO fileVo = zgFileService.getFile(dto.getFileId());
        File file = zgFileService.transferToLocal(dto.getFileId());
        // 拿到文件后进行分片然后再分片上传
        // 获取 分片总数
        Integer blockCount = (int) Math.ceil(file.length()*1.0 / chunkSize);
        String uploadId = IdUtil.fastUUID();
        try {
            FileInputStream fileInputStream = new FileInputStream(file);
            byte[] bytes = new byte[chunkSize];
            int start = 0;
            int chunkNumber =0;
            while ((start = fileInputStream.read(bytes)) != -1) {
                String name2 = fileVo.getFilenameWithExt();
                String filename =  Paths.get(fsProperties.getRootDir()).toAbsolutePath().normalize()+"/"+CHUNK_TMP_CATEGORY+"/"+fileVo.getFilename() + "-" + chunkNumber + name2.substring(name2.lastIndexOf("."));
                File file1 = new File(filename);
                FileOutputStream outputStream = new FileOutputStream(file1);
                outputStream.write(bytes,0, start);
                outputStream.flush();
                outputStream.close();
                GisResultVO<Boolean> model_compress_file = gisFileFeignClient.uploadChunk(MultipartFileUtils.toMultipartFile(file1), "MODEL_COMPRESS_FILE", chunkNumber, uploadId, blockCount, uploadId, fileVo.getFilename());
                log.info("上传状态:{},chunkNumber{},uploadId:{},blockCount:{}",getResultData(model_compress_file),chunkNumber,uploadId,blockCount);
                file1.delete();
                chunkNumber++;
            }
  
            return ModelGisResourceVO.builder().build();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return ModelGisResourceVO.builder().build();
    }
```

