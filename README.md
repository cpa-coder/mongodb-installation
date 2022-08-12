# MongoDB Installation Setup for Windows

Configure MongoDB server with authentication and replica set.

## Table of Contents

- [MongoDB Installation Setup for Windows](#mongodb-installation-setup-for-windows)
  - [Table of Contents](#table-of-contents)
  - [Software Application Requirements](#software-application-requirements)
  - [Step-by-step Instruction](#step-by-step-instruction)
  - [Database Backup and Restore](#database-backup-and-restore)
  - [Disclaimer](#disclaimer)

## Software Application Requirements

1. [OpenSSL v3](https://www.mediafire.com/file/sjjrxjs072jgm98/Win64OpenSSL_Light-3_0_0.exe)
2. [MongoDB Server Community Edition](https://www.mongodb.com/try/download/community)
3. [Mongo Shell](https://www.mongodb.com/try/download/shell)


## Step-by-step Instruction

1. Install all the [required applications](#application-requirements). Make sure to check if environment variables are properly setup/installed to run `mongod`, `mongosh` and `openssl` from command-line.

2. Now lets enable replica set.

   Open `mongod.cfg` file in your favorite text editor and edit the replica set section
    ```yaml
    replication:
      replSetName: "[name]"
    ```
   ***Note:***
    You may set any name for `replSetName` property.

   Restart MongoDB server.

   Run a termimal and type following command to enable replica set.
    ```bash
    #run mongo shell without authentication
    mongosh

    #to initiate replica set
    rs.initiate()
    ```


3. Create key file. This key file is use to allow authentication with replica set.
   
   Suppose you want to create the key to "C:\keyfile" directory with "key.pem" key file name, run the following commands:

    ```bash
    mkdir C:\keyfile
    
    openssl rand -base64 756 > C:\keyfile\key.pem

    icacls.exe C:\keyfile\key.pem /reset
    icacls.exe C:\keyfile\key.pem /GRANT:R "$($env:USERNAME):(R)"
    icacls.exe C:\keyfile\key.pem /inheritance:r
    ```

    ***Note:*** You may change filename and file path location as much as you like.

4. Run `mongo shell` to add new user with admin previlege with the following command:

    ```bash
    mongosh

    use admin
    
    db.createUser(
      {
        user:'[user]',
        pwd:'[password]',
        roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
      }
    )
    ```

    ***Note:*** Change user and password as needed.

    To check newly added user, run the following commands:

    ```bash
    use admin
    
    db.getUsers()
    ```

   To add user for specific database, run the following command:

    ```bash
    use [database]
    db.createUser(
      {
        user:'[user]',
        pwd:'[password]',
        roles: [ { role: "readWrite", db: '[database]'} ]
      }
    )
    ```

5. Now its time to enable authentication.
   
    Copy the key file from Step <a href="#step-by-step-instruction">3</a> to where mongod.exe is located i.e. `C:\Program Files\MongoDB\Server\[version]\bin`.

    Open `mongod.cfg` using text editor with admin previledge and update the security section:

    ```yaml
    security:
      authorization: enabled
      keyFile: C:\Program Files\MongoDB\Server\[version]\bin\key.pem

   

6. If you install MongoDB as a service without authentication, remove the existing service by running this command

    ```bash
    mongod --remove
    ```
7. Then it is necessary to create a new service for MongoDB

    ```bash
    mongod --journal --config "C:\Program Files\MongoDB\Server\[version]\bin\mongod.cfg" --dbpath "C:\Program Files\MongoDB\Server\[version]\data" --auth --install
    ``` 

8. Make sure MongoDB service is running

9.  Once MongoDB service is up running, open a terminal and run `mongo shell` with the following commands:

    ```bash
    #Supply user and password setup on step 6.
    mongosh -u [user]

    #Then enter user password
    ```

10. Congratulations!!! You have successfully configure MongoDB Server :tada::clap::clap::clap::clap:

---

## Database Backup and Restore

Make sure you install [MongoDb Database Tools](https://www.mongodb.com/try/download/database-tools)

- Backup MongoDB database

    To back MongoDB database use the following command:

    ```bash
    #To create directory
    mkdir "C:\back-up"
    cd "C:\back-up"
 
    #To backup database
    mongodump -u [user] --authenticationDatabase admin --db [database name] 

    #Then enter user password
    ```

    You can also specified output location with "`--out=[output folder path]`".

    ***Note:*** When output directory is not specified backup database will be created in "`.\dump\[database name]`" folder.

- Restore MongoDB database

    To restore MongoDB database use the following command

    ```bash
    mongorestore -u [user] --authenticationDatabase admin --db [database name] [database folder path]

    #Then enter user password
    ```
## Disclaimer

This is created for personal use. In no event shall the author be liable for any claim, damages or other liability in connection with this instruction. ***Use this at your OWN RISK.***:warning:
