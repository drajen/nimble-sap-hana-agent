## HPE Nimble Storage Application Snapshot Agent for SAP HANA

The HPE Nimble Storage Application Snapshot Agent for SAP HANA uses the [HPE Nimble Storage Snapshot Framework](https://infosight.hpe.com/InfoSight/media/cms/active/pubs_Nimble_Snapshot_Framework_NOS_4x.whz/index.html) (requires HPE InfoSight login) to support application consistent and integrated storage level snapshots of SAP HANA. 

The workflow for the HPE Nimble Storage Application Snapshot Agent for SAP HANA has 3 phases :

1. **Pre-Snapshot Operation**

   In this phase, Nimble OS communicates with the agent to execute a hdbsql command to create a savepoint in the SAP HANA database. Once the savepoint is created, the agent then queries the SAP HANA database to the savepoint identifier. The agent then waits for 60 seconds to allow SAP HANA to write metadata into the file system 

2. **Storage snapshot creation on Nimble OS**

   Once the pre-snapshot phase completes, Nimble OS takes the snapshot of all the volumes that form the application storage (consistency group).

3. **Post-Snapshot Operation**

   Nimble OS communicates with the agent again by passing information related to the snapshot completion If the snapshot was successful, the snapshot ID is sent to the agent. The agent then executes another hdbsql command to confirm the backup was successful. If the snapshot failed, the agent will execute a hdbsql statement to that effect.

## Implementation details
This implementation of the Agent is in Java with Jetty 8.1 as the servlet container for the REST server.

- `AgentService.java` has the main method to start the server.
- `SnapshotTaskResourceImpl.java` and `AgentResourceImpl.java` classes implement the REST APIs as defined in the documentation for the Nimble Snapshot Framework.
  These classes are where you would add the code for the `preSnapshot` and `postSnapshot` tasks.
- To build and run the application, you need Java and `gradle` installed on your dev machine.
- Gradle will use maven central repository to pull down some dependent libraries, so your dev machine needs to be connected to the internet.

## Building the agent
Steps to import the project, build and run:

1. Download and install Java 8 and latest version of gradle
   https://gradle.org/install
2. Set `GRADLE_HOME` and `JAVA_HOME` environment variables to point to the installation directories of Java and Gradle respectively
3. If you would like to import the project in eclipse, then from the root directory of the repository, run `gradle eclipse`
This generates the files required for the project to be imported into eclipse.
4. To build, run `gradle build`
5. To run the application via gradle, execute `gradle run`

## Package
After compilation, the deliverable will be located in build/distributions as a zip file.
Uncompressed the deliverable folder would look like this:
```
nimble-backup-agent-ri
↳ bin
| ↳ nimble-backup-agent-ri
| ↳ sap-hana-backup-agent-config.xml
↳ lib
  ↳ .jar files
```

## Installation
To install the SAP HANA backup agent, perform the following steps:

1. Unzip the deliverable file to your run location on the host machine. This will be referred to as the AGENT-DIR in subsequent steps.

2. Edit the `sap-hana-backup-agent-config.xml` file in `AGENT-DIR/bin` and enter the correct configuration information.

3. Locate the `ngdbc.jar` file on the SAP HANA host. Use the following command:
   ```
   find / -name ngdbc.jar 2> /dev/null
   ```
   
4. Verify that you have a `.bashrc` file in your home directory. If not, create one and add the following line:
   ```
   #!/bin/bash
   ```

5. Set the `SAP_JDBC_DRIVER` environment variable to point to the `ngdbc.jar` file. The easiest way to do this is to modify the `.bashrc` file by adding the following line:
   ```
   export SAP_JDBC_DRIVER=/PATH]/TO/ngdbc.jar
   ```
   This will ensure that the environment variable for the backup agent user is set at login.

6. To trigger the `.bashrc` to run do:
   ```
   source .bashrc
   ```

7. To launch the backup agent:
   ```
   cd AGENT-DIR/bin
   ./nimble-sap-agent
   ```

## License
This software is licensed under the Apache License version 2.0. Please see the [LICENSE](LICENSE) file for full terms and conditions.
