Here is a detailed, step-by-step guide to setting up, building, and running the API Factory Backend in IntelliJ IDEA based on the provided solution.

Step 1: Create a New Project
Open IntelliJ IDEA.

Click New Project.

Name: factory-backend

Location: Choose a folder on your computer.

Language: Java

Build system: Maven

JDK: Select 17 (or higher). If you don't have it, click "Download JDK" and select version 17.

GroupId: com.apifactory

ArtifactId: factory-backend

Click Create.

Step 2: Configure Dependencies (pom.xml)
In the Project Explorer (left sidebar), locate and open pom.xml in the root folder.

Delete the existing content and replace it entirely with the pom.xml content found in Part 1, Section 1 of the reference document.

Crucial Step: Locate the floating Maven icon (usually appears in the top-right of the code editor) that says "Load Maven Changes" (or press Ctrl+Shift+O / Cmd+Shift+O). Click it to download the dependencies (spring-boot-starter-web, lombok, h2, mustache, etc.).

Step 3: Create Java Package Structure
Navigate to src > main > java.

Right-click on java and select New > Package.

Enter the name: com.apifactory.

Inside com.apifactory, create the following sub-packages (Right-click com.apifactory > New > Package):

controller

model

repository

service

Step 4: Create Java Classes
Copy the code from the reference document into files in the corresponding packages:

Main Application:

Right-click com.apifactory > New > Java Class.

Name: BackendApplication.

Paste content from Part 1, Section 2 (BackendApplication.java).

Model:

Right-click com.apifactory.model > New > Java Class.

Name: ApiProject.

Paste content from Part 1, Section 2 (ApiProject.java).

Repository:

Right-click com.apifactory.repository > New > Java Class.

Name: ProjectRepository (Select "Interface" from the list if prompted, or just paste the code over).

Paste content from Part 1, Section 2 (ProjectRepository.java).

Service:

Right-click com.apifactory.service > New > Java Class.

Name: GeneratorEngine.

Paste content from Part 1, Section 2 (GeneratorEngine.java).

Repeat for: ProjectService (Paste ProjectService.java).

Controller:

Right-click com.apifactory.controller > New > Java Class.

Name: FactoryController.

Paste content from Part 1, Section 2 (FactoryController.java).

Step 5: Configure Resources & Templates
Navigate to src > main > resources.

Application Properties:

Open application.properties.

Paste the content from Part 1, Section 1.

Templates Directory:

Right-click on resources > New > Directory.

Name: templates.

Create Mustache Files:

Right-click on the new templates folder > New > File.

Create the following 5 files exactly as named and paste the corresponding content from Part 2 of the reference document:

root-pom.xml.mustache

contract-pom.xml.mustache

service-pom.xml.mustache

Application.java.mustache

ApiDelegateImpl.java.mustache

Step 6: Run the Application
Open src/main/java/com/apifactory/BackendApplication.java.

Look for the green "Play" arrow icon next to the public class BackendApplication or the main method line numbers.

Click it and select Run 'BackendApplication...'.

Wait for the console logs at the bottom. You should see Started BackendApplication in X seconds.

Step 7: Verify
The backend is now running on http://localhost:8080.

Test with Postman/Curl:

Create Draft: POST to http://localhost:8080/api/projects with body {"name": "Test", "groupId": "com.test", "artifactId": "demo", "version": "1.0.0"}.

Upload Swagger: PUT http://localhost:8080/api/projects/{UUID}/swagger (form-data: file).

Generate: POST http://localhost:8080/api/projects/{UUID}/generate.

Download: GET http://localhost:8080/api/projects/{UUID}/download.

Connect the UI:

If you have the React UI running (e.g., via npm start on port 3000), it will now successfully connect to this backend because the Controller has @CrossOrigin(origins = "*").