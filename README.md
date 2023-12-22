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

