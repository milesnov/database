# Setting up the Environment

## Introduction

*Estimated Lab Time*: 30 minutes (TEMP)

### About Product/Technology

### Objectives
- Provision an Autonomous Database (ADB)
- Create and associate Users and Groups within Identity and Access Management (IAM)
- Connect to the provisioned ADB

### Prerequisites
This lab assumes you have:
- A Free Tier, Paid or LiveLabs Oracle Cloud account
- You have completed:
    - Lab: Prepare Setup (*Free-tier* and *Paid Tenants* only)
    - Lab: Environment Setup
    - Lab: Initialize Environment



## Task 1: Provision the Autonomous Database

1. Identify the root compartment to create the ADB and IAM policy in.

    ```
    export ROOT_COMP_ID=`oci iam compartment list --include-root --raw-output --query "data[?contains(\"id\",'tenancy')].id | [0]"`
    ```

2. Name your ADB

    ```
    export DB_NAME=lltest
    ```

3. Create your password for the database

    ```
    export ADMIN_PWD=Oracle123+Oracle123+
    ```

4. Verify your username within IAM. This is to make sure you are signed into the correct account, one that has the privileges to create the users.

    ```
    export OCI_USER_NAME=`oci iam user list --raw-output --query "data[?contains(\"id\",'"${OCI_CS_USER_OCID}"')].name| [0]"`
    echo $OCI_USER_NAME
    ```

5. Create the ADB with the name and password we decided on earlier.

    ```
    oci db autonomous-database create --compartment-id $ROOT_COMP_ID --db-name ${DB_NAME} --display-name ${DB_NAME} --is-free-tier true  --admin-password $ADMIN_PWD --cpu-core-count 1 --data-storage-size-in-tbs 1
    ```

6. Allow any user in the tenancy to access the ADB.

    ```
    oci iam policy create  --name grant-adb-access --compartment-id $ROOT_COMP_ID  --statements '[ "allow any-user to use autonomous-database-family in tenancy"]' --description 'policy for granting any user to access autonomous databases'
    ```

## Task 2: Create and assign test user and groups

1. Create a test user Debra in IAM (COMMENT: May need to alter to also be assigned an email to work)

    ```
    oci iam user create --name DBA_DEBRA --description "User Account for DEBRA the DBA"
    ```

2. Create two groups in IAM, one for all database users and one for database admins.

    ```
    oci iam group create --name ALL_DB_USERS --description "Group for all of the DB Users"
    ```
    ```
    oci iam group create --name DB_ADMIN --description "Group for DB Admins"
    ```

3. Setup and verify environment variables for ease of use in commands later.

    ```
    export ADB_OCID=`oci db autonomous-database list --compartment-id $ROOT_COMP_ID --raw-output --query "data[?contains(\"db-name\",'lltest')].id | [0]"`
    echo $ADB_OCID
    ```
    ```
    export DB_ADMIN_OCID=`oci iam group list --raw-output --query "data[?contains(\"name\",'DB_ADMIN')].id | [0]"`
    echo $DB_ADMIN_OCID
    ```
    ```
    export ALL_DB_USERS_OCID=`oci iam group list --raw-output --query "data[?contains(\"name\",'ALL_DB_USERS')].id | [0]"`
    echo $ALL_DB_USERS_OCID
    ```
    ```
    export DBA_DEBRA_OCID=`oci iam user list --raw-output --query "data[?contains(\"name\",'DBA_DEBRA')].id | [0]"`
    echo $DBA_DEBRA_OCID
    ```

4. Add your test user Debra to both groups that were created earlier.

    ```
    oci iam group add-user --user-id $DBA_DEBRA_OCID --group-id $ALL_DB_USERS_OCID
    ```
    ```
    oci iam group add-user --user-id $DBA_DEBRA_OCID --group-id $DB_ADMIN_OCID
    ```

5. Add your cloud shell user to the ALL_DB_USERS group

    ```
    oci iam group add-user --user-id $OCI_CS_USER_OCID --group-id $ALL_DB_USERS_OCID
    ```

## Task 3: Connect to the Autonomous Database
