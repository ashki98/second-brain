# Jenkins & CI/CD Walkthrough

Reference: YT Video: https://www.youtube.com/watch?v=mk2FBuTMwDc

# **CI/CD Pipeline Using Jenkins | Continuous Integration & Continuous Deployment | DevOps | Simplilearn**

The YouTube video "CI/CD Pipeline Using Jenkins | Continuous Integration & Continuous Deployment | DevOps | Simplilearn" explains the concepts of CI/CD pipelines using Jenkins and their implementation in DevOps.

Here's a summary of the key concepts and diagrams discussed:

**Core Concepts:**

- **DevOps:** DevOps is the integration of development and operations practices to streamline software development and deployment. It emphasizes collaboration, automation, and faster release cycles. DevOps aims to improve release frequency, team collaboration, management, and issue resolution.
    - **Diagram:** The video includes a diagram [[00:50](http://www.youtube.com/watch?v=mk2FBuTMwDc&t=50)] that visually represents the integration of development and operations in DevOps. It illustrates development teams focusing on coding, testing, and product performance, while operations teams manage infrastructure, deployments, and maintenance.
- **CI/CD Pipeline:** A CI/CD pipeline is the backbone of DevOps, automating the process of delivering code changes to production. It consists of a sequence of automated steps, including building, testing, and deployment.
    - **Continuous Integration (CI):** CI is the practice of regularly integrating code changes from multiple developers into a shared repository. This process includes automated build and testing to detect bugs early in the development cycle.
    - **Continuous Delivery (CD):** CD prepares code changes for deployment to customers. It focuses on automating the deployment process to reduce manual errors and increase efficiency, enabling faster and more reliable releases.
    - **Benefits:** CI/CD enables faster and more reliable software releases, improves team agility, and increases confidence in deployments.
- **Jenkins:** Jenkins is an open-source automation server widely used for implementing CI/CD pipelines. It is flexible, offers a wide range of plugins, and supports various programming languages, making it a versatile tool for automating the entire development process.

**Building a CI/CD Pipeline with Jenkins:**

- **Diagram**: A diagram [[14:20](http://www.youtube.com/watch?v=mk2FBuTMwDc&t=860)] outlines the six key steps to build a CI/CD pipeline in Jenkins:
    1. **Set up Java JDK:** Java Development Kit is a prerequisite for running Jenkins.
    2. **Download Jenkins:** Download the Jenkins WAR (Web Application Archive) file.
    3. **Create a pipeline job:** Set up a new pipeline job within Jenkins.
    4. **Configure pipeline scripts:** Configure the pipeline script, either directly within Jenkins or by linking to a script in a Source Code Management (SCM) system like Git.
    5. **Execute the pipeline:** Run the pipeline to automate the build, test, and deployment stages.
    6. **Monitor the build process:** Track the pipeline's execution and monitor logs for status and potential issues.
- **Demo of CI/CD Pipeline in Jenkins:** The video demonstrates building a CI/CD pipeline in Jenkins, showcasing the stages involved, including checkout, build, tests, SonarQube analysis, artifact archiving, deployment, and notifications.
    - **Jenkins UI Screenshots**: The demo includes screenshots of the Jenkins user interface [[21:43](http://www.youtube.com/watch?v=mk2FBuTMwDc&t=1303)] to illustrate the practical steps of creating a new job, configuring pipeline settings, linking a Git repository, and initiating a build. The UI demonstrates how to set up and monitor a CI/CD pipeline in Jenkins.
