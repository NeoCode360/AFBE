API Factory Backend - Complete Project Source

This document contains all the necessary files to build the API Factory Backend in IntelliJ IDEA.

Prerequisites:

Java 17+

Maven 3.8+

1. Project Configuration

File: pom.xml

Location: Root of the project

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="[http://maven.apache.org/POM/4.0.0](http://maven.apache.org/POM/4.0.0)" 
         xmlns:xsi="[http://www.w3.org/2001/XMLSchema-instance](http://www.w3.org/2001/XMLSchema-instance)"
         xsi:schemaLocation="[http://maven.apache.org/POM/4.0.0](http://maven.apache.org/POM/4.0.0) [https://maven.apache.org/xsd/maven-4.0.0.xsd](https://maven.apache.org/xsd/maven-4.0.0.xsd)">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>
    
    <groupId>com.apifactory</groupId>
    <artifactId>factory-backend</artifactId>
    <version>1.0.0</version>
    <name>API Factory Backend</name>
    <description>Backend service for generating Spring Boot Microservices</description>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <!-- Web API -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- Persistence (JPA + H2) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Boilerplate reduction -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <!-- Templating Engine (Spring Boot Starter) -->
        <!-- Automatically configures Mustache.Compiler bean -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mustache</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>


File: application.properties

Location: src/main/resources/application.properties

server.port=8080
spring.h2.console.enabled=true
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect

# File Upload Limits
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB


2. Java Source Code

Package: com.apifactory

File: BackendApplication.java
Location: src/main/java/com/apifactory/BackendApplication.java

package com.apifactory;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class BackendApplication {
    public static void main(String[] args) {
        SpringApplication.run(BackendApplication.class, args);
    }
}


Package: com.apifactory.model

File: ApiProject.java
Location: src/main/java/com/apifactory/model/ApiProject.java

package com.apifactory.model;

import jakarta.persistence.*;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;
import java.util.UUID;

@Entity
@Table(name = "api_projects")
@Data
@NoArgsConstructor
public class ApiProject {

    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;

    // --- Metadata ---
    private String name;
    private String groupId;
    private String artifactId;
    private String version;
    private String description;

    @Enumerated(EnumType.STRING)
    private ProjectStatus status = ProjectStatus.DRAFT;

    // --- Configuration ---
    @ElementCollection(fetch = FetchType.EAGER)
    @CollectionTable(name = "project_features", joinColumns = @JoinColumn(name = "project_id"))
    @Column(name = "feature_name")
    private List<String> enabledFeatures = new ArrayList<>();

    // --- Assets ---
    @Lob
    @Column(columnDefinition = "CLOB") 
    private String openApiSpec; // The uploaded Swagger JSON

    @Lob
    @Column(columnDefinition = "BLOB") 
    private byte[] businessLogicJar; // The uploaded Logic JAR

    @Lob
    @Column(columnDefinition = "BLOB") 
    private byte[] generatedSourceZip; // The final artifact

    // --- Audit ---
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    @PrePersist
    protected void onCreate() { createdAt = LocalDateTime.now(); updatedAt = LocalDateTime.now(); }
    @PreUpdate
    protected void onUpdate() { updatedAt = LocalDateTime.now(); }

    public enum ProjectStatus { DRAFT, ASSETS_UPLOADED, CONFIGURED, GENERATED }
}


Package: com.apifactory.repository

File: ProjectRepository.java
Location: src/main/java/com/apifactory/repository/ProjectRepository.java

package com.apifactory.repository;

import com.apifactory.model.ApiProject;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import java.util.UUID;

@Repository
public interface ProjectRepository extends JpaRepository<ApiProject, UUID> {
}


Package: com.apifactory.service

File: GeneratorEngine.java
Location: src/main/java/com/apifactory/service/GeneratorEngine.java

package com.apifactory.service;

import com.apifactory.model.ApiProject;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.samskivert.mustache.Mustache;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.ByteArrayResource;
import org.springframework.core.io.ClassPathResource;
import org.springframework.stereotype.Component;

import java.io.*;
import java.nio.charset.StandardCharsets;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;

@Component
public class GeneratorEngine {

    // Spring Boot provides the configured Compiler bean
    private final Mustache.Compiler compiler;
    private final ObjectMapper objectMapper;

    @Autowired
    public GeneratorEngine(Mustache.Compiler compiler, ObjectMapper objectMapper) {
        this.compiler = compiler;
        this.objectMapper = objectMapper;
    }

    public ByteArrayResource generateProjectZip(ApiProject project) throws IOException {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        
        // Use try-with-resources to ensure ZIP stream closes correctly
        try (ZipOutputStream zos = new ZipOutputStream(baos)) {
            
            // 1. Build Context
            Map<String, Object> context = buildContext(project);
            String basePkgPath = ((String) context.get("basePackage")).replace(".", "/");
            
            // --- 1. ROOT MODULE ---
            addFile(zos, "pom.xml", render("templates/root-pom.xml.mustache", context));

            // --- 2. CONTRACT MODULE ---
            // Generates POJOs and Interfaces via plugin
            addFile(zos, "contract/pom.xml", render("templates/contract-pom.xml.mustache", context));
            addFile(zos, "contract/src/main/resources/openapi.json", project.getOpenApiSpec());

            // --- 3. SERVICE MODULE ---
            // Main Spring Boot Application
            addFile(zos, "service/pom.xml", render("templates/service-pom.xml.mustache", context));
            
            // Application Entry Point
            String mainClassPath = "service/src/main/java/" + basePkgPath + "/Application.java";
            addFile(zos, mainClassPath, render("templates/Application.java.mustache", context));

            // Delegate Implementation
            String implPath = "service/src/main/java/" + basePkgPath + "/controller/" + context.get("implClassName") + ".java";
            addFile(zos, implPath, render("templates/ApiDelegateImpl.java.mustache", context));

            // Scaffold folders for Service and Model
            addFile(zos, "service/src/main/java/" + basePkgPath + "/service/package-info.java", "package " + context.get("basePackage") + ".service;");
            addFile(zos, "service/src/main/java/" + basePkgPath + "/model/package-info.java", "package " + context.get("basePackage") + ".model;");

            // Optional: Inject Logic JAR
            if (project.getBusinessLogicJar() != null) {
                addFileBytes(zos, "service/lib/business-logic.jar", project.getBusinessLogicJar());
            }
            
            // Dockerfile
            addFile(zos, "Dockerfile", "FROM openjdk:17-jdk-slim\nCOPY service/target/*.jar app.jar\nENTRYPOINT [\"java\",\"-jar\",\"/app.jar\"]");
        }
        
        return new ByteArrayResource(baos.toByteArray());
    }

    private Map<String, Object> buildContext(ApiProject p) {
        Map<String, Object> ctx = new HashMap<>();
        ctx.put("groupId", p.getGroupId());
        ctx.put("artifactId", p.getArtifactId());
        ctx.put("version", p.getVersion());
        ctx.put("basePackage", p.getGroupId() + "." + p.getArtifactId().replace("-", ""));
        ctx.put("hasIdempotency", p.getEnabledFeatures().contains("idempotency"));
        ctx.put("hasValidation", p.getEnabledFeatures().contains("validation"));
        
        // Predict the Delegate Interface name
        String delegateName = determineDelegateName(p.getOpenApiSpec());
        ctx.put("delegateInterface", delegateName);
        ctx.put("implClassName", delegateName + "Impl");
        return ctx;
    }

    private String determineDelegateName(String openApiSpec) {
        try {
            if (openApiSpec == null || openApiSpec.isEmpty()) return "ApiDelegate";
            JsonNode root = objectMapper.readTree(openApiSpec);
            JsonNode paths = root.path("paths");
            if (paths.isObject()) {
                Iterator<JsonNode> elements = paths.elements();
                if (elements.hasNext()) {
                    JsonNode path = elements.next();
                    Iterator<JsonNode> methods = path.elements();
                    while (methods.hasNext()) {
                        JsonNode method = methods.next();
                        if (method.has("tags") && method.get("tags").isArray() && method.get("tags").size() > 0) {
                            String tag = method.get("tags").get(0).asText();
                            return capitalize(tag) + "ApiDelegate";
                        }
                    }
                }
            }
        } catch (Exception e) { /* Fallback */ }
        return "ApiDelegate";
    }

    private String capitalize(String str) {
        if (str == null || str.isEmpty()) return str;
        return str.substring(0, 1).toUpperCase() + str.substring(1);
    }

    private String render(String templatePath, Map<String, Object> context) throws IOException {
        ClassPathResource resource = new ClassPathResource(templatePath);
        if (!resource.exists()) throw new IOException("Template not found: " + templatePath);
        try (Reader reader = new InputStreamReader(resource.getInputStream(), StandardCharsets.UTF_8)) {
            return compiler.compile(reader).execute(context);
        }
    }

    private void addFile(ZipOutputStream zos, String path, String content) throws IOException {
        zos.putNextEntry(new ZipEntry(path));
        zos.write(content.getBytes(StandardCharsets.UTF_8));
        zos.closeEntry();
    }
    
    private void addFileBytes(ZipOutputStream zos, String path, byte[] content) throws IOException {
        zos.putNextEntry(new ZipEntry(path));
        zos.write(content);
        zos.closeEntry();
    }
}


File: ProjectService.java
Location: src/main/java/com/apifactory/service/ProjectService.java

package com.apifactory.service;

import com.apifactory.model.ApiProject;
import com.apifactory.repository.ProjectRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.core.io.ByteArrayResource;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.multipart.MultipartFile;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.UUID;

@Service
@RequiredArgsConstructor
public class ProjectService {

    private final ProjectRepository repository;
    private final GeneratorEngine generatorEngine;

    public ApiProject getProject(UUID id) {
        return repository.findById(id).orElseThrow(() -> new RuntimeException("Project not found"));
    }

    @Transactional
    public ApiProject createProject(ApiProject project) {
        return repository.save(project);
    }

    @Transactional
    public void uploadSwagger(UUID id, MultipartFile file) throws IOException {
        ApiProject project = getProject(id);
        project.setOpenApiSpec(new String(file.getBytes(), StandardCharsets.UTF_8));
        project.setStatus(ApiProject.ProjectStatus.ASSETS_UPLOADED);
        repository.save(project);
    }

    @Transactional
    public void uploadJar(UUID id, MultipartFile file) throws IOException {
        ApiProject project = getProject(id);
        project.setBusinessLogicJar(file.getBytes());
        repository.save(project);
    }

    @Transactional
    public ApiProject updateConfig(UUID id, ApiProject updates) {
        ApiProject project = getProject(id);
        project.setEnabledFeatures(updates.getEnabledFeatures());
        project.setStatus(ApiProject.ProjectStatus.CONFIGURED);
        return repository.save(project);
    }

    @Transactional
    public void generateAndPersist(UUID id) throws IOException {
        ApiProject project = getProject(id);
        if (project.getOpenApiSpec() == null) throw new IllegalStateException("Swagger Spec missing");

        ByteArrayResource zip = generatorEngine.generateProjectZip(project);
        
        project.setGeneratedSourceZip(zip.getByteArray());
        project.setStatus(ApiProject.ProjectStatus.GENERATED);
        repository.save(project);
    }

    @Transactional(readOnly = true)
    public byte[] getArtifact(UUID id) {
        ApiProject project = getProject(id);
        if (project.getGeneratedSourceZip() == null) throw new RuntimeException("Not generated yet");
        return project.getGeneratedSourceZip();
    }
}


Package: com.apifactory.controller

File: FactoryController.java
Location: src/main/java/com/apifactory/controller/FactoryController.java

package com.apifactory.controller;

import com.apifactory.model.ApiProject;
import com.apifactory.service.ProjectService;
import lombok.RequiredArgsConstructor;
import org.springframework.core.io.ByteArrayResource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import java.io.IOException;
import java.util.UUID;

@RestController
@RequestMapping("/api/projects")
@CrossOrigin(origins = "*") 
@RequiredArgsConstructor
public class FactoryController {

    private final ProjectService service;

    @PostMapping
    public ResponseEntity<ApiProject> create(@RequestBody ApiProject project) {
        return ResponseEntity.ok(service.createProject(project));
    }

    @PutMapping(value = "/{id}/swagger", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public ResponseEntity<Void> uploadSwagger(@PathVariable UUID id, @RequestParam("file") MultipartFile file) throws IOException {
        service.uploadSwagger(id, file);
        return ResponseEntity.ok().build();
    }

    @PutMapping(value = "/{id}/jar", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public ResponseEntity<Void> uploadJar(@PathVariable UUID id, @RequestParam("file") MultipartFile file) throws IOException {
        service.uploadJar(id, file);
        return ResponseEntity.ok().build();
    }

    @PutMapping("/{id}/config")
    public ResponseEntity<ApiProject> configure(@PathVariable UUID id, @RequestBody ApiProject updates) {
        return ResponseEntity.ok(service.updateConfig(id, updates));
    }

    @PostMapping("/{id}/generate")
    public ResponseEntity<Void> generate(@PathVariable UUID id) throws IOException {
        service.generateAndPersist(id);
        return ResponseEntity.ok().build();
    }

    @GetMapping("/{id}/download")
    public ResponseEntity<ByteArrayResource> download(@PathVariable UUID id) {
        byte[] data = service.getArtifact(id);
        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment;filename=api-project.zip")
                .contentType(MediaType.APPLICATION_OCTET_STREAM)
                .contentLength(data.length)
                .body(new ByteArrayResource(data));
    }
}


3. Templates (Target API)

Location: src/main/resources/templates/ (Create this folder)

File: root-pom.xml.mustache

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="[http://maven.apache.org/POM/4.0.0](http://maven.apache.org/POM/4.0.0)" xmlns:xsi="[http://www.w3.org/2001/XMLSchema-instance](http://www.w3.org/2001/XMLSchema-instance)" xsi:schemaLocation="[http://maven.apache.org/POM/4.0.0](http://maven.apache.org/POM/4.0.0) [https://maven.apache.org/xsd/maven-4.0.0.xsd](https://maven.apache.org/xsd/maven-4.0.0.xsd)">
    <modelVersion>4.0.0</modelVersion>
    <groupId>{{groupId}}</groupId>
    <artifactId>{{artifactId}}-root</artifactId>
    <version>{{version}}</version>
    <packaging>pom</packaging>
    
    <modules>
        <module>contract</module>
        <module>service</module>
    </modules>
    
    <properties>
        <java.version>17</java.version>
        <spring.boot.version>3.2.0</spring.boot.version>
    </properties>
</project>


File: contract-pom.xml.mustache

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="[http://maven.apache.org/POM/4.0.0](http://maven.apache.org/POM/4.0.0)" xmlns:xsi="[http://www.w3.org/2001/XMLSchema-instance](http://www.w3.org/2001/XMLSchema-instance)" xsi:schemaLocation="[http://maven.apache.org/POM/4.0.0](http://maven.apache.org/POM/4.0.0) [https://maven.apache.org/xsd/maven-4.0.0.xsd](https://maven.apache.org/xsd/maven-4.0.0.xsd)">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>{{groupId}}</groupId>
        <artifactId>{{artifactId}}-root</artifactId>
        <version>{{version}}</version>
    </parent>
    
    <artifactId>{{artifactId}}-contract</artifactId>
    <packaging>jar</packaging>

    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-web</artifactId><version>3.2.0</version></dependency>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-validation</artifactId><version>3.2.0</version></dependency>
        <dependency><groupId>org.openapitools</groupId><artifactId>jackson-databind-nullable</artifactId><version>0.2.6</version></dependency>
        <dependency><groupId>io.swagger.core.v3</groupId><artifactId>swagger-annotations</artifactId><version>2.2.15</version></dependency>
        <dependency><groupId>org.projectlombok</groupId><artifactId>lombok</artifactId><version>1.18.30</version><optional>true</optional></dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.openapitools</groupId>
                <artifactId>openapi-generator-maven-plugin</artifactId>
                <version>7.1.0</version>
                <executions>
                    <execution>
                        <goals><goal>generate</goal></goals>
                        <configuration>
                            <inputSpec>${project.basedir}/src/main/resources/openapi.json</inputSpec>
                            <generatorName>spring</generatorName>
                            <output>${project.build.directory}/generated-sources</output>
                            <apiPackage>{{basePackage}}.api</apiPackage>
                            <modelPackage>{{basePackage}}.model</modelPackage>
                            <generateApiTests>false</generateApiTests>
                            <generateModelTests>false</generateModelTests>
                            <globalProperties><skipFromModel>False</skipFromModel></globalProperties>
                            <configOptions>
                                <delegatePattern>true</delegatePattern>
                                <useTags>true</useTags>
                                <useJakartaEe>true</useJakartaEe>
                                <dateLibrary>java8</dateLibrary>
                                <additionalModelTypeAnnotations>@lombok.Builder @lombok.Data @lombok.NoArgsConstructor @lombok.AllArgsConstructor</additionalModelTypeAnnotations>
                            </configOptions>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>


File: service-pom.xml.mustache

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="[http://maven.apache.org/POM/4.0.0](http://maven.apache.org/POM/4.0.0)" xmlns:xsi="[http://www.w3.org/2001/XMLSchema-instance](http://www.w3.org/2001/XMLSchema-instance)" xsi:schemaLocation="[http://maven.apache.org/POM/4.0.0](http://maven.apache.org/POM/4.0.0) [https://maven.apache.org/xsd/maven-4.0.0.xsd](https://maven.apache.org/xsd/maven-4.0.0.xsd)">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>{{groupId}}</groupId>
        <artifactId>{{artifactId}}-root</artifactId>
        <version>{{version}}</version>
    </parent>
    
    <artifactId>{{artifactId}}-service</artifactId>
    
    <dependencies>
        <dependency>
            <groupId>{{groupId}}</groupId>
            <artifactId>{{artifactId}}-contract</artifactId>
            <version>{{version}}</version>
        </dependency>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-web</artifactId><version>3.2.0</version></dependency>
        {{#hasValidation}}
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-validation</artifactId><version>3.2.0</version></dependency>
        {{/hasValidation}}
        {{#hasIdempotency}}
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-data-redis</artifactId><version>3.2.0</version></dependency>
        {{/hasIdempotency}}
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>3.2.0</version>
            </plugin>
        </plugins>
    </build>
</project>


File: Application.java.mustache

package {{basePackage}};

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}


File: ApiDelegateImpl.java.mustache

package {{basePackage}}.controller;

import {{basePackage}}.api.*;
import {{basePackage}}.model.*;
import org.springframework.stereotype.Service;
import org.springframework.http.ResponseEntity;

/**
 * Service implementation for the API Delegate.
 * Implements the interface generated in the 'contract' module.
 */
@Service
public class {{implClassName}} implements {{delegateInterface}} {
    
    public ResponseEntity<String> healthCheck() {
        return ResponseEntity.ok("API Service is running");
    }
}
