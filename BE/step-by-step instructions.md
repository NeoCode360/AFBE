Here are the detailed, step-by-step instructions to set up the API Factory Backend code from the Canvas in your local IntelliJ IDEA.

Step 1: Create a New Project
Open IntelliJ IDEA.

Click New Project (or File > New > Project).

Name: factory-backend

Location: Choose a directory on your machine.

Language: Java

Build system: Maven

JDK: Select version 17 (or higher).

GroupId: com.apifactory

ArtifactId: factory-backend

Click Create.

Step 2: Configure Dependencies (pom.xml)
In the Project view (left sidebar), locate and open the pom.xml file in the root directory.

Delete the existing content.

Copy the entire content from Section 1 (File: pom.xml) in the Canvas document and paste it into your pom.xml.

Crucial: Click the Load Maven Changes button (a small floating 'M' icon with a refresh symbol) that appears in the top-right of the editor, or right-click pom.xml > Maven > Reload Project. This downloads the necessary libraries (Spring Boot, Lombok, etc.).

Step 3: Create Java Package Structure
Navigate to src > main > java > com > apifactory. (If only java exists, right-click java > New > Package > com.apifactory).

Right-click on the com.apifactory package and create the following sub-packages:

model

repository

service

controller

Step 4: Create Java Classes
Copy the code from Section 2 of the Canvas into the corresponding packages:

Main Application:

Right-click com.apifactory > New > Java Class > Name: BackendApplication.

Paste the content for BackendApplication.java.

Model:

Right-click com.apifactory.model > New > Java Class > Name: ApiProject.

Paste the content for ApiProject.java.

Repository:

Right-click com.apifactory.repository > New > Java Class > Name: ProjectRepository.

Paste the content for ProjectRepository.java.

Service:

Right-click com.apifactory.service > New > Java Class > Name: GeneratorEngine.

Paste the content for GeneratorEngine.java.

Right-click com.apifactory.service > New > Java Class > Name: ProjectService.

Paste the content for ProjectService.java.

Controller:

Right-click com.apifactory.controller > New > Java Class > Name: FactoryController.

Paste the content for FactoryController.java.

Step 5: Configure Resources
Navigate to src > main > resources in the Project view.

Application Properties:

Open the existing application.properties file.

Paste the content from Section 1 (File: application.properties).

Templates:

Right-click on the resources folder > New > Directory.

Name it: templates.

Inside the templates directory, create the following 5 files (Right-click templates > New > File) and paste the corresponding content from Section 3 of the Canvas:

root-pom.xml.mustache

contract-pom.xml.mustache

service-pom.xml.mustache

Application.java.mustache

ApiDelegateImpl.java.mustache

Step 6: Build and Run
Open src/main/java/com/apifactory/BackendApplication.java.

Look for the green Play (Run) arrow next to the public class BackendApplication line or the main method.

Click it and select Run 'BackendApplication.main()'.

Watch the Run console at the bottom. Wait until you see a log message similar to: Tomcat started on port 8080 (http) Started BackendApplication in X.XXX seconds

Step 7: Verify
Your API Factory Backend is now running locally!

Base URL: http://localhost:8080

Test Endpoint: You can verify it is running by checking the H2 console at http://localhost:8080/h2-console (JDBC URL: jdbc:h2:mem:testdb, Password: password).