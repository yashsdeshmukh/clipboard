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

@RunWith(SpringRunner.class)
@WebMvcTest(MyController.class)
public class MyControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ProcessBuilder processBuilder; // Mock ProcessBuilder

    @Test
    public void testDownloadFile() throws Exception {
        // Mock the ProcessBuilder behavior
        Process processMock = Mockito.mock(Process.class);
        Mockito.when(processMock.waitFor()).thenReturn(0); // Mock successful command execution

        Mockito.when(processBuilder.start()).thenReturn(processMock);

        // Perform the GET request
        MvcResult mvcResult = mockMvc.perform(MockMvcRequestBuilders.get("/api/downloadFile"))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.header().string(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=file.txt"))
                .andReturn();

        // Assert that the response contains the expected content
        MockHttpServletResponse response = mvcResult.getResponse();
        Assert.assertNotNull(response.getContentAsString());

        // You may add more assertions based on your specific requirements
    }
}

@Test
    public void testDownloadFile() throws Exception {
        // Mock the ProcessBuilder behavior
        Process processMock = Mockito.mock(Process.class);
        Mockito.when(processMock.waitFor()).thenReturn(0); // Mock successful command execution
        Mockito.when(processBuilder.start()).thenReturn(processMock);

        // Mock the file path and content for the UrlResource
        Path filePath = Paths.get("/path/to/created/file.txt");
        String fileContent = "Mock file content";
        String fileName = "file.txt";
        UrlResource urlResourceMock = Mockito.mock(UrlResource.class);
        Mockito.when(urlResourceMock.getFilename()).thenReturn(fileName);
        Mockito.when(urlResourceMock.getInputStream()).thenReturn(new ByteArrayInputStream(fileContent.getBytes()));
        Mockito.when(urlResourceMock.exists()).thenReturn(true);

        // Mock the UrlResource creation
        Mockito.when(new UrlResource(filePath.toUri())).thenReturn(urlResourceMock);

        // Perform the GET request
        MvcResult mvcResult = mockMvc.perform(MockMvcRequestBuilders.get("/api/downloadFile"))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.header().string(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=" + fileName))
                .andReturn();

        // Assert that the response contains the expected content
        MockHttpServletResponse response = mvcResult.getResponse();
        Assert.assertEquals(fileContent, response.getContentAsString());

        // You may add more assertions based on your specific requirements
    }


Service Invoker - 
import java.util.List;
import java.util.concurrent.CompletableFuture;

public class ServiceInvoker {
    public static void main(String[] args) {
        String inputFilePath = "input.txt";
        String outputFilePath = "output.txt";
        int batchSize = 1000; // Adjust batch size based on your requirements

        try {
            CompletableFuture<List<String>> readFuture = Reader.readLinesAsync(inputFilePath);
            readFuture.thenApply(lines -> processBatches(lines, batchSize))
                    .thenCompose(batches -> CompletableFuture.allOf(
                            batches.stream()
                                    .map(batch -> batch.thenCompose(Parser::parseLinesAsync)
                                            .thenCompose(Transformer::transformLinesAsync)
                                            .thenCompose(transformedLines -> Writer.writeLinesAsync(transformedLines, outputFilePath)))
                                    .toArray(CompletableFuture[]::new)))
                    .thenAccept(result -> System.out.println("Processing complete."))
                    .exceptionally(e -> {
                        e.printStackTrace();
                        return null;
                    })
                    .join();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static List<CompletableFuture<List<String>>> processBatches(List<String> lines, int batchSize) {
        List<CompletableFuture<List<String>>> batches = new ArrayList<>();
        for (int i = 0; i < lines.size(); i += batchSize) {
            List<String> batch = lines.subList(i, Math.min(i + batchSize, lines.size()));
            batches.add(CompletableFuture.completedFuture(batch));
        }
        return batches;
    }
}


Writer - 
import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;
import java.util.List;
import java.util.concurrent.CompletableFuture;

public class Writer {
    public static CompletableFuture<Void> writeLinesAsync(List<String> transformedLines, String filePath) {
        return CompletableFuture.runAsync(() -> {
            try (BufferedWriter writer = new BufferedWriter(new FileWriter(filePath))) {
                for (String line : transformedLines) {
                    writer.write(line);
                    writer.newLine();
                }
            } catch (IOException e) {
                throw new RuntimeException("Error writing file", e);
            }
        });
    }
}

Transformer - 
import java.util.List;
import java.util.concurrent.CompletableFuture;

public class Transformer {
    public static CompletableFuture<List<String>> transformLinesAsync(List<String[]> parsedLines) {
        return CompletableFuture.supplyAsync(() -> {
            List<String> transformedLines = new ArrayList<>();
            for (String[] tokens : parsedLines) {
                StringBuilder transformedLine = new StringBuilder();
                for (String token : tokens) {
                    // Assuming token format is "tag=value"
                    String[] parts = token.split("=");
                    String tag = parts[0];
                    String value = parts[1];
                    // Transformation logic based on tag
                    if (tag.equals("35")) {
                        // Example transformation for tag 35
                        value = "NEW_" + value;
                    }
                    transformedLine.append(tag).append("=").append(value).append("^A");
                }
                transformedLines.add(transformedLine.toString());
            }
            return transformedLines;
        });
    }
}

Parser - 

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CompletableFuture;

public class Parser {
    public static CompletableFuture<List<String[]>> parseLinesAsync(List<String> lines) {
        return CompletableFuture.supplyAsync(() -> {
            List<String[]> parsedLines = new ArrayList<>();
            for (String line : lines) {
                String[] tokens = line.split("\u0001");
                parsedLines.add(tokens);
            }
            return parsedLines;
        });
    }
}

Reader -

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CompletableFuture;

public class Reader {
    public static CompletableFuture<List<String>> readLinesAsync(String filePath) {
        return CompletableFuture.supplyAsync(() -> {
            List<String> lines = new ArrayList<>();
            try (BufferedReader reader = new BufferedReader(new FileReader(filePath))) {
                String line;
                while ((line = reader.readLine()) != null) {
                    lines.add(line);
                }
            } catch (IOException e) {
                throw new RuntimeException("Error reading file", e);
            }
            return lines;
        });
    }
}


import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.ByteArrayDeserializer;
import org.apache.avro.generic.GenericRecord;
import org.apache.avro.io.DatumReader;
import org.apache.avro.io.Decoder;
import org.apache.avro.io.DecoderFactory;
import org.apache.avro.specific.SpecificDatumReader;
import org.apache.avro.Schema;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;
import java.util.Scanner;

public class KafkaAvroConsumer {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        while (true) {
            // User input for Kafka topic and schema registry URL
            System.out.print("Enter Kafka topic (or 'exit' to quit): ");
            String topic = scanner.nextLine();
            if (topic.equalsIgnoreCase("exit")) {
                break;
            }
            System.out.print("Enter Avro schema (as JSON string): ");
            String schemaString = scanner.nextLine();
            System.out.print("Enter rowkey value to filter: ");
            String rowkeyFilter = scanner.nextLine();

            // Parse the provided Avro schema
            Schema schema = new Schema.Parser().parse(schemaString);

            // Set Kafka consumer properties
            Properties properties = new Properties();
            properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");  // Adjust if needed
            properties.put(ConsumerConfig.GROUP_ID_CONFIG, "avro-consumer-group");
            properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, ByteArrayDeserializer.class.getName());
            properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, ByteArrayDeserializer.class.getName());

            // Create Kafka consumer
            KafkaConsumer<byte[], byte[]> consumer = new KafkaConsumer<>(properties);

            // Subscribe to the topic
            consumer.subscribe(Collections.singletonList(topic));

            // Poll and process records
            try {
                final long maxIdleTimeMillis = 30000;  // Adjust the idle time as needed
                long lastPollTime = System.currentTimeMillis();

                while (true) {
                    ConsumerRecords<byte[], byte[]> records = consumer.poll(Duration.ofMillis(100));

                    if (records.isEmpty()) {
                        if (System.currentTimeMillis() - lastPollTime > maxIdleTimeMillis) {
                            System.out.println("No new records for " + (maxIdleTimeMillis / 1000) + " seconds. Exiting...");
                            break;
                        }
                    } else {
                        lastPollTime = System.currentTimeMillis();

                        for (ConsumerRecord<byte[], byte[]> record : records) {
                            // Check the header for rowkey
                            String rowkey = record.headers().lastHeader("rowkey") != null
                                    ? new String(record.headers().lastHeader("rowkey").value())
                                    : null;

                            if (rowkey != null && rowkey.equals(rowkeyFilter)) {
                                // Deserialize the Avro record
                                DatumReader<GenericRecord> datumReader = new SpecificDatumReader<>(schema);
                                Decoder decoder = DecoderFactory.get().binaryDecoder(record.value(), null);
                                GenericRecord avroRecord = datumReader.read(null, decoder);

                                System.out.println("Filtered Record: " + avroRecord);
                            }
                        }
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                consumer.close();
            }
        }
        scanner.close();
    }
}


