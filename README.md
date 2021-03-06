# End of Module Assignment - Queens Medical Centre ASMIS Secure Design


## Table of Contents
1. [Introduction](#introduction)
2. [Secure Authentication](#secure-authentication)
3. [Tracing Capabilities via Log Database](#tracing-capabilities-via-log-database)
4. [Simple Staff Interface](#simple-staff-interface)
5. [Prerequisites](#prerequisites)
6. [MySQL Database](#mysql-database)
7. [MySQL Connection in Python Code and Security Measures](#mysql-connection-in-python-code-and-security-measures)
8. [Python Modules](#python-modules)
9. [Folder Structure](#folder-structure)
10. [Testing Process and Data](#testing-process-and-data)
11. [Functional Test Plan](#functional-test-plan)
12. [Reference List](#reference-list)



### Introduction
This project serves as the end of module assignment for Launching into Cyber Security September 2020. The aim of this project is to implement some of the solutions recommended as part of the report submitted in Unit 9 as Python code.

In total, the submitted code for the end of module assignment aims to implement three major parts:
1.	Secure Authentication and Password Hashing
2.	Tracing Capabilities via Log Database for Search and Edit Operations
3.	Simple Staff Interface to Demo Implementations



### Secure Authentication
**Threat:**
Spoofing describes attack vectors in which the attacker impersonates another user, a process or a system. In the case of the ASMIS, a likely case of a spoofing attack that may occur is the attacker authenticating with another user's credentials. This may happen via brute force or, if the user uses the same password elsewhere, having been obtained via an external database leak.

**Solution:**
To reduce the risk of having a cyber criminal obtain system access via another user's credentials, it is vital to enforce strict password conventions. Having a combination of a minimum password length and complexity restrictions in place often makes brute-forcing the password very difficult due to the extended time required. Increasing the password complexity to an 8-character full alpha-numeric password increases the required time to brute force it to more than 252 days at 10 million attempts per second (Cheswick, 2012). Additionally, the password should never be stored as a plain text string; instead, the password should be encrypted upon registration and stored as a hash to compare against when a login attempt is made.

The python implementation includes the initial sign-up process with accepting the user-submitted password and only storing its hash in a MySQL database. Upon login, the entered password will be compared against the hash and thus will determine if the user has authenticated with acceptable credentials.



### Tracing Capabilities via Log Database
**Threat:**
Repudiation describes a user denying to have acted without other parties having the means to prove otherwise. This can occur if the system has no way of tracing performed actions of users. For instance, a specialist might illegitimately access a patient's file not currently under consultation. Without traces or logs, it would not be possible to prove the specialist has done so. This would also mean that an information disclosure threat has occurred. The information has been exposed to a party who should not have access to it.

**Solution:**
Tracing all actions conducted by each user on the system via log files is an integral part of a secure implementation. Nonrepudiation is essential to account for cases where data is illegitimately accessed or tampered with by malicious staff. By having a log created whenever a patient file is created, viewed or changed, it is possible to trace back the event to whoever actioned it.

For the sake of this implementation, a log database will be created. The python code will also include some sample functions such as: searchPatient(), editPatient(), searchAppointment(), and editAppointment() to demo the logging functionality. Every time any one of these operations is executed, a log will be created in the database containing the following information:
-	Date and time stamp
-	Type of operation (Search / Edit)
- User ID who executed the operation
-	Effected data table (Patient / Appointment)
- Attribute that was searched for or edited (e.g. "first name")
- If search operation: exact search term that was entered
- If edit operation: record in question (ID of either patient or appointment)
- If edit operation: value prior to edit (old value) and value after edit (new value)



### Simple Staff Interface
The project furthermore includes a simple staff interface which allows for staff members to sign in, register, search and also edit data (appointments and patients). This interface serves as a primary mean to demo the project implementation and test the security aspects of this implementation.

This document will later highlight a number of test plans that have been conducted via the interface and which can be replicated on your own when running the project.



### Prerequisites
Prior to running the python project, you will need to install a few external libraries:
- Argon2 v1.3 for password hashing via the “argon2-cffi” python lib [https://github.com/hynek/argon2-cffi]
- Fernet symmetric encryption to encrypt and decrypt MySQL credential via .bin file [https://cryptography.io/en/latest/fernet.html] 
- Official MySQL connector lib to connect to the database [https://dev.mysql.com/doc/connector-python/en/] 

You can use the following pip commands to install the libraries directly:
```
pip install argon2-cffi
pip install cryptography
pip install mysql-connector-python
```

This project has been coded in python 3.6.8 and it is recommended to use the same version when executing the code.

![Python Version](https://i.imgur.com/i0SiBVt.png)



### MySQL Database 
The project uses three main databases for the implementation:

1. **Authentication**
2. **Data**
3. **Eventlog**

**Authentication:**
Authentication contains the user logins and their respective passwords as an Argon2 hash. When registering a new user, a new data row is inserted into the 'logins' table containing the staff's desired username, the chosen password as a hash, and the staff's first and last name.

When logging into an already existing account, the user needs to input a username and password. Once done, the entered username is compared against all entries in the table. If it exists, the entered password is hashed and the hash itself is compared against the saved entry. If the hash and the username both line up with the existing data, the user is successfully authenticated in the system and can continue.

*'authentication.logins' table columns:*
![logins table columns](https://i.imgur.com/dNgbqc7.png)

*'authentication.logins' table sample data:*
![logins data table](https://i.imgur.com/eLJQCrc.png)


**Data:**
The 'data' database contains the main patient and appointment information that is saved in the system. The information is respectively saved in the two tables, 'patients' and 'appointments'. Patient or appointment information that is searched for in the system is queried against the appropriate table and displayed back to the user. If a user actions an edit, the appropriate data row is updated on this database.

*'data.patients' table columns:*
![patient table columns](https://i.imgur.com/aXRb7Yi.png)

*'data.patients' table sample data:*
![patients data table](https://i.imgur.com/jXxpCpb.png)

*'data.appointments' table columns:*
![appointments table columns](https://i.imgur.com/vE1uWOl.png)

*'data.appointments' table sample data:*
![appointments data table](https://i.imgur.com/UV6UkvI.png)


**Eventlog:**
The eventlog database serves as the log database that stores all traces of user operations on the system. The table 'events' stores logs for edit and search actions conducted via the staff interface. For all actions the data includes when the action was conducted, what action it is (edit/search), on what table is was executed (patients/appointments), who actioned it (user id), and what attribute was effected (e.g. 'firstname'). For search actions it further includes the user inputed search term. For edit actions it includes the record that was edited, the old value prior to the change and the new value after the change.

*'eventlog.events' table columns:*
![events table columns](https://i.imgur.com/G2vmvIz.png)

*'eventlog.events' table sample data:*
![events data table](https://i.imgur.com/fHOB8jz.png)



### MySQL Connection in Python Code and Security Measures
For the MySQL database insert, update, and read operations, a database user is required. For this project, the user 'client'@'localhost' was created which is used for all interface operations. Prior to executing any operations, the python code needs to authenticate with the database. Instead of adding the MySQL username and password into the code as plain text, I have opted to encrypt the credentials and store them as a binary file in the 'config' folder for security purposes (using Fernet to encrypt). The encryption script and clear text credentials prior to encryption can be viewed in the 'setup' folder. When running the project, the 'dbconnection' module will attempt to decrypt the credentials binary file using the credentials.bin file stored in the same 'config' folder. Only if that is successful, the interface will be able to operate (search, edit, login, register). In an actual deployment, you would be able to limit the 'config' folder access controls, so that only the code is able to access the folder and read the 'credentials.bin' and 'key.bin' files, not the user himself. This would make it difficult for the user to obtain the actual credentials and connect to the database directly.

Another layer of security is provided by the application of the least privilege principle. The user 'client'@'localhost' is only granted privileges that are vital for the interface operations. These are mainly select (for search and login), insert (for registration) and update (for edit) rights for the specific data tables. This ensures that in case of a breach or SQL injection attack using this user, the data that may be effected is limited. A full list of granted privileges for the user can be seen here:

!['client'@'localhost' privileges](https://i.imgur.com/rBKvXBn.png)

SQL injection attacks are a big point for security concerns and are still regarded as the most critical web application security risk according to the OWASP Top Ten list (OWASP, 2020). The same is valid for this python project as well. The staff interface is asking for user inputs, which - if no security measures are taken - can be manipulated to execute such injections. All the malicious user would have to do, for instance, is to escape the actual username select query and insert an attack in its place, e.g.:
```
input("Please enter your username: ") -> "'; select * from logins where id = 1; --"
```

This is a huge problem, due to the attacker being able to authenticate as any user if no measures are taken. This projects employs two precautions to circumvent such an attack:

1. For all SQL queries that are tied to a user input, the query parameter values are escaped. This means that whenever the MySQL execute function is called, the values are not just added into the query string and passed as such, but instead are passed as a separate argument. This ensures that if a malicious injection is attempted, it does not execute.

2. For all user inputs, the inputed values are validated through a 'sanitizeInput' function in the operations module. Only if the input meets the given constraints, the input is accepted and passed back to the handler function. Again, this ensures that no special characters can be passed where they are not expected.



### Python Modules
In total, the ASMIS project includes a total of five python modules:
1. **interface.py:** main staff interface that includes handler functions to call the other modules

2. **dbconnection.py:** module that decrypts the MySQL credentials and establishes the primary connection to the db

3. **authentication.py:** module that handles the login and registration operations, as well as the password hashing functionality

4. **operations.py:** module that handles the main portions of the search and edit operations, as well as sanitization of inputs 

5. **eventlog.py:** module that handles the event logging portion of the project

For the execution of the actual program (staff interface), please run ```python interface.py```.



### Folder Structure
The python modules are all located on the root folder of the project. Furthermore, the project includes two additional folders: 'config' and 'setup'.

**Config:**

Config includes three binary files that are used for the code execution. In an actual deployment, this folder would need to enforce strict access controls to protect the sensitive files inside of it:
1. **banner.bin:** file that contains the initial lines to be printed when the interface is executed

2. **credentials.bin:** file that contains the encrypted MySQL credentials (encrypted via Fernet)

3. **key.bin:** file that contains the key for the Fernet encryption and which is used to decrypt the credentials when the code is executed


**Setup:**

Setup includes files to understand the initial setup of the project. In an actual deployment, this folder would not exist. In total, the folder includes three files:
1. **clearinput.bin:** displays the MySQL credentials prior to being encrypted (format: "host:user:password:database")

2. **encrypt.py:** the python script used to initially encrypt the clear credentials and outputs the credentials.bin file as seen in the 'config' folder

3. **sampledata.sql:** initial sql script used to add sample data to both the 'data.patients' and the 'data.appointments' tables



### Testing Process and Data
To test the implementation on your end, you will need to execute the 'interface.py' script. This functions as a basic demo interface for staff members. With it, you will be able to register, login, search for patient and appointment data, as well as edit the data. For each edit and search operation done via the system, an event log will be created, which can be viewed in the 'eventlog.events' MySQL data table. You will be able to create an own user once you boot up the interface. I have created a demo user account which can be used as well: 

```
username: essex-uni
password: Pso$bTM7a6G2
```

For sample patient and appointment data to search for and edit, please see provided sample data in setup folder. This should be the default data existent in the system prior to any edits made.

For a full list of test scenarios, please see the list below under "Functional Test Plan".



### Functional Test Plan
**1. Register functionality**
- [✓] Username may only include alpha numerical values and special characters ```. _ -```, and must have at least 3 characters
- [✓] Password needs to include letters and numbers and be at least 8 characters long
- [✓] 'Password' and 'Confirm Password' inputs need be the same
- [✓] Entered first and last name may only include letters, spaces and ```-```, and must have at least 2 characters
- [✓] If username already exists, prompt error
- [✓] Once created, password should be stored as hash in the 'authentication.logins' table

**2. Sign in functionality**
- [✓] Sign in works for existing user (e.g. 'essex-uni' given in the Testing Data section)
- [✓] Sign in works for newly registered user
- [✓] Entering a non-existent username should throw up an error
- [✓] Entering a wrong password should throw up an error
- [✓] If sign in is successful, should greet staff user by first name

**3. Patient search and subsequent log creation**
- [✓] Name searches (first+last) should only accept letters, spaces and ```-```, and must have at least 2 characters
- [✓] Date of birth searches should only accept numbers and ```-```
- [✓] First name search works as expected + creates appropriate log
- [✓] Last name search works as expected + creates appropriate log
- [✓] Date of birth search works as expected + creates appropriate log

**4. Patient edit and subsequent log creation**
- [✓] New name value (first+last) should only accept letters, spaces and ```-```, and must have at least 2 characters
- [✓] New date of birth values should only accept numbers and ```-```
- [✓] First name edit works as expected + creates appropriate log
- [✓] Last name edit works as expected + creates appropriate log
- [✓] Date of birth edit works as expected + creates appropriate log

**5. Appointment search and subsequent log creation**
- [✓] Date searches should only accept numbers and ```-```
- [✓] Id (patient and staff) should only accept integers
- [✓] Date search works as expected + creates appropriate log
- [✓] Patient id search works as expected + creates appropriate log
- [✓] Consulting staff id search works as expected + creates appropriate log

**6. Appointment edit and subsequent log creation**
- [✓] Date and time edits should only accept numbers and ```-```
- [✓] Consulting staff id edits should only accept integers
- [✓] Date edit works as expected + creates appropriate log
- [✓] Time edit works as expected + creates appropriate log
- [✓] Consulting staff id edit works as expected + creates appropriate log

**7. General**
- [✓] Main menu works as expected (log out, patient search, appointment search)
- [✓] "Return to main menu" and "Cancel" options work as expected



### Reference List
Cheswick, W. (2012) Rethinking Passwords. Association for Computing Machinery (ACM) 10(12). Available from: https://queue.acm.org/detail.cfm?id=2422416 [Accessed 9 December 2020].

OWASP (2020) OWASP Top Ten. Available from: https://owasp.org/www-project-top-ten/ [Accessed 9 December 2020].