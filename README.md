# 🚀 Node.js CI Pipeline with Azure DevOps

This repository contains an **Azure DevOps pipeline** that automates the build and packaging process for **Node.js applications**.  
It takes care of installing dependencies, archiving required files, and publishing build artifacts that can be used in **release pipelines** or downloaded for deployment.

---

## 📌 What This Pipeline Does

1. **Trigger on Branch** – Automatically runs when code is pushed to a specific branch.  
2. **Checkout Code** – Pulls the repository code into the pipeline.  
3. **Install Dependencies** – Runs `npm install` to install Node.js packages.  
4. **Copy Files** – Collects `.js` and `.json` files for packaging.  
5. **Archive Files** – Compresses files into a `.zip` for portability.  
6. **Publish Artifacts** – Uploads the archive to the pipeline’s **Artifacts** section for deployment.

---

## 📂 Full Pipeline YAML

```yaml
trigger:
  branches: 
    include:
      - __BRANCH_NAME__     # Replace with your working branch (e.g. 'main')

# Pipeline run naming format: Date + revision
name: $(date:yyyyMMdd)$(rev:.r)

jobs:
- job: Job_1
  displayName: Agent job 1
  pool:
    vmImage: 'ubuntu-latest'   # Hosted agent VM image
  steps:
    # Step 1: Get source code
    - checkout: self
      fetchDepth: 1

    # Step 2: Install Node.js dependencies
    - task: Npm@1
      displayName: npm install
      inputs:
        workingDir: ./
        verbose: false

    # Step 3: Copy necessary files
    - task: CopyFiles@2
      displayName: 'Copy Files'
      inputs:
        SourceFolder: ./
        Contents: |
          **\*.js
          **\*.json
        TargetFolder: ./output

    # Step 4: Archive files into a .zip
    - task: ArchiveFiles@2
      displayName: Archive files
      inputs:
        rootFolderOrFile: ./output
        includeRootFolder: false
        archiveType: zip
        archiveFile: $(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip
        replaceExistingArchive: true

    # Step 5: Publish the archived artifact
    - task: PublishBuildArtifacts@1
      displayName: 'Publish artifacts: drop'
      inputs:
        PathtoPublish: $(Build.ArtifactStagingDirectory)
        ArtifactName: 'drop'
        publishLocation: 'Container'
