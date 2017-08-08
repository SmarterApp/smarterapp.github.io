## Create an AWS RDS Instance
* Choose an Relational Database Service (RDS) instance
* Select either **MySQL** or **Amazon Aurora** as the database engine
  * For an installation that will support a large number of concurrent students (e.g. more than 100,000) Aurora is recommended
* Specify the instance details:
  * Select instance type
  * Provide instance identifier
  * Provide user name and password
* Configure network and security options to comply with your institutions standards/practices
* After the RDS instance has been created create a new **Parameter Group** and set the following parameter to **true**:
  * `log_bin_trust_function_creators`