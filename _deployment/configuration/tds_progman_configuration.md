---
title: TDS Progman Configuration
permalink: "deployment/configuration/tds_progman_configuration.html"
layout: "document"
categories: ["student", "tds-config"]
---

## Configuring Progman for TDS
Progman is used to hold configuration information for TDS_Student and TDS_Proctor Web applications.  They both use the same configuration and are referenced in the deployment documentation for the `PROGMAN_LOCATOR` property in the tds-student-war.yml and the tds-proctor-war.yml.

At the end of this document is a sample progman configuration that you will need to alter to fit the deployment.  Anywhere with `<CHANGE ME>` should be altered with your specific deployment information.

## Steps to create new progman configuration
These are general steps to create a new progman configuration.  If you want to alter a configuration you can skip step two and three.

1. Once logged into Progman go to `Configure Component Properties` link.
2. Click on the `New` button if you want to create a new configuration otherwise skip this step.
3. Select a **name** and **environment**.  Usually the name is `tds` but you can pick whatever you'd like.
4. Select the `Property File Entry` radio button entry.
5. Paste the property sample below in the section.
6. Now you can either alter the `<CHANGE ME>` elements in this mode or switch back to `Form Editor` mode.
7. Once done save the configuration.

### Sample TDS Configuration

```
tds.testshell.dictionaryUrl=http://<CHANGE ME>:8080/dictionary
tds.content.remote.url=http://tds-content-service/
performance.datasource.idleTimeout=120000
datasource.idleTimeout=120000
tds.assessment.remote.url=http://tds-assessment-service/
tds.exam.legacy.enabled=false
tds.exam.remote.url=http://tds-exam-service/exam
tds.exam.remote.enabled=true
tds.session.legacy.enabled=false
tds.session.remote.enabled=true
tds.session.remote.url=http://tds-session-service/sessions
student.ItembankDBName=itembank
student.TDSConfigsDBName=configs
student.TDSSessionDBName=session
student.TDSArchiveDBName=archive
component.name=Proctor
datasource.acquireIncrement=5
datasource.acquireRetryAttempts=5
datasource.checkoutTimeout=60000
datasource.driverClassName=com.mysql.jdbc.Driver
datasource.idleConnectionTestPeriod=14400
datasource.maxConnectionAge=0
datasource.maxPoolSize=24
datasource.maxStatements=20000
datasource.maxStatementsPerConnection=100
datasource.minPoolSize=12
datasource.numHelperThreads=3
datasource.password=<CHANGE ME>
datasource.testConnectionOnCheckin=false
datasource.testConnectionOnCheckout=false
datasource.url=jdbc:mysql://<CHANGE ME>/session
datasource.username=remoteuser
dbLockRetryAttemptMax=500
dbLockRetrySleepInterval=116
EncryptionKey=Thisisanincrediblylongkeythatiscertainlylongerthantwentyfourcharacters
iris.ContentPath=/usr/local/tomcat/content/
itemScoring.callbackUrl=http://requestb.in/16hqwk61
itemscoring.qti.sympyMaxTries=3
itemscoring.qti.sympyServiceUrl=http://tds-equation-scoring-service/
itemscoring.qti.sympyTimeoutMillis=10000
logLatencyInterval=55
logLatencyMaxTime=30000
mna.logger.level=ERROR
mna.mnaUrl=http://<mna-context-url>/mna-rest/
mna.oauth.client.id=mna
mna.oauth.client.secret=<CHANGE ME>
mnaNodeName=dev
mnaServerName=proctor_dev
oauth.testreg.client.granttype=<CHANGE ME>
oauth.testreg.client.secret=<CHANGE ME>
oauth.testreg.client=pm
oauth.testreg.client.id=pm
oauth.testreg.password=<CHANGE ME>
oauth.testreg.username=<CHANGE ME>
oauth.tsb.client.secret=<CHANGE ME>
oauth.tsb.client=tsb
opportunity.isScoredByTDS=false
permission.security.profile=dev
permission.uri=<CHANGE ME>
proctor.Appkey=Proctor
proctor.AppName=Proctor
proctor.ClientName=SBAC_PT
proctor.ClientQueryString=false
proctor.DBJndiName=java:/comp/env/jdbc/session
proctor.Debug.AllowFTP=true
proctor.EncryptedPassword=true
proctor.IsCheckinSite=false
proctor.ItembankDBName=itembank
proctor.oauth.checktoken.endpoint=<CHANGE ME>
proctor.oauth.resource.client.id=pm
proctor.oauth.resource.client.secret=<CHANGE ME>
proctor.RecordSystemClient=true
proctor.security.idp=<CHANGE ME>
proctor.SessionType=0
proctor.SqlCommandTimeout=60
proctor.StateCode=CA
proctor.TDSArchiveDBName=archive
proctor.TDSConfigsDBName=configs
proctor.TDSSessionDBName=session
proctor.TestRegistrationApplicationUrl=<CHANGE ME>
student.Appkey=Student
student.AppName=Student
student.ClientCookie=true
student.ClientName=SBAC_PT
student.ClientQueryString=true
student.DBDialect=MYSQL
student.Debug.AllowFTP=true
student.EncryptedPassword=true
student.IsCheckinSite=false
student.oauth.resource.client.secret=<CHANGE ME>
student.RecordSystemClient=true
student.SessionType=0
student.SqlCommandTimeout=60
student.StateCode=CA
student.StudentMaxOpps=2
student.TestRegistrationApplicationUrl=<CHANGE ME>
student.testScoring.logDebug=true
student.testScoring.logError=true
performance.datasource.maxPoolSize=24
performance.datasource.minPoolSize=12
performance.logMaxTestOpportunities.enabled=false
performance.logLatency.enabled=false
```


[back to TDS Configuration Tasks](index.html)