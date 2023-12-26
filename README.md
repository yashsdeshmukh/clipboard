# clipboard

1.
    private final String fileDirectory = "/path/to/your/files"; // Replace with the actual path to your files

    @GetMapping("/{fileName}")
    public ResponseEntity<Resource> downloadFile(@PathVariable String fileName) throws MalformedURLException {
        // Construct the file path
        Path filePath = Paths.get(fileDirectory).resolve(fileName);
        
        // Load the file as a resource
        Resource resource = new UrlResource(filePath.toUri());

        // Check if the file exists
        if (resource.exists() || resource.isReadable()) {
            return ResponseEntity.ok()
                    .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=" + resource.getFilename())
                    .contentType(MediaType.APPLICATION_OCTET_STREAM)
                    .body(resource);
        } else {
            return ResponseEntity.notFound().build();
        }

2.
public class FileController {

    private final String fileDirectory = "/path/to/your/files"; // Replace with the actual path to your files

    @GetMapping("/{fileName}")
    public String readFileContents(@PathVariable String fileName) throws IOException {
        // Construct the file path
        Path filePath = Paths.get(fileDirectory).resolve(fileName);

        // Read file contents into a string
        String fileContents = new String(Files.readAllBytes(filePath));

        return fileContents;
    }
}

public class FileUploadController {

    @PostMapping("/upload")
    @ApiOperation(value = "Upload a file")
    public ResponseEntity<String> handleFileUpload(@RequestParam("file") MultipartFile file) {
        try {
            // Specify the target directory
            Path targetDirectory = Path.of("path/to/your/target/directory");

            // Check if the target directory exists, create it if not
            if (!Files.exists(targetDirectory)) {
                Files.createDirectories(targetDirectory);
            }

            // Resolve the target path by combining the target directory with the file name
            Path targetPath = targetDirectory.resolve(file.getOriginalFilename());

            // Copy the file to the target directory, replacing existing file
            Files.copy(file.getInputStream(), targetPath, StandardCopyOption.REPLACE_EXISTING);

            // Return a success message
            return ResponseEntity.ok("File uploaded successfully at: " + targetPath);
        } catch (IOException e) {
            e.printStackTrace();
            return ResponseEntity.status(500).body("Error uploading the file");
        }
    }
}

