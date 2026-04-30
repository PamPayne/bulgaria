---
source_url: https://dbnsa.sharepoint.com/sites/ComplianceOfficers93/Regulatory Repository/Rest of World/Bulgaria/2. Analysis/Dictao_D3S_JEL_APIGettingStartedGuide_WSBG_v1 0en.pdf
country: Bulgaria
document_name: Dictao_D3S_JEL_APIGettingStartedGuide_WSBG_v1 0en.pdf
source_file: Dictao_D3S_JEL_APIGettingStartedGuide_WSBG_v1 0en.pdf
source_url: https://dbnsa.sharepoint.com/sites/ComplianceOfficers93/Regulatory Repository/Rest of World/Bulgaria/2. Analysis/Dictao_D3S_JEL_APIGettingStartedGuide_WSBG_v1 0en.pdf
extracted_date: 2026-04-30
jurisdiction: Bulgaria
description: Developer getting started guide for integrating with Dictao D3S web services for Bulgarian gaming regulatory compliance.
regulatory_body: State Commission on Gambling
---

# D3S Developer's Getting Started Guide

## Web Services Integration - Bulgaria

D3S Gaming Safe 5.0

D3S Developer's Getting Started Guide  
Web Services Integration - Bulgaria  
Ref. : Dictao_D3S_JEL_APIGettingStartedGuide_WSBG_v1.0en Version 1.0 2014/10/02

Communication, reproduction and use restricted to the purpose of this document

Reference : Dictao_D3S_JEL_APIGettingStartedGuide_WSBG_v1.0en  
Version : 1.0  
Date of latest update : 2014/10/02  
Related application version : 5.0  
Confidentiality : Non

## Revision list

| Version n° | Status (1) | Date | Author | Reason for / nature of update |
|---|---|---|---|---|
| 0.01 | P | 2014/09/12 | Dictao | Creation |
| 1.0 | V | 2014/10/02 | Dictao | Completed, reviewed, and approved. |

(1) P : in progress ; V : Validated

## Summary

1 Introduction .................................................................................... 4  
1.1 Special note for Bulgaria ............................................................................... 4  
2 Principles........................................................................................ 5  
2.1 Architecture .................................................................................................... 5  
2.2 Environments ................................................................................................. 5  
2.3 Web Service connection................................................................................ 5  
2.4 WSDL convention .......................................................................................... 6  
2.4.1 WSDL definition ....................................................................................................................... 6  
2.4.2 WSDL example ........................................................................................................................ 7  
2.5 XSD convention ............................................................................................. 7  
2.5.1 XSD definitions ......................................................................................................................... 7  
2.5.2 XSD example ........................................................................................................................... 8  
2.6 Exception management ................................................................................. 9  
2.6.1 UserFaultException .................................................................................................................. 9  
2.6.2 SystemFaultException ........................................................................................................... 10  
2.7 Records passed as a list in all methods .....................................................10  
2.8 Asynchronous calls return value ................................................................10  
2.9 Idempotence ..................................................................................................10  
2.9.1 Principles ................................................................................................................................ 10  
2.9.2 Guidelines .............................................................................................................................. 11  
3 Additional Documentation .......................................................... 12

## 1 Introduction

The D3S software is a secure storage module enabling gaming applications to store records of gaming events. Typically, such records are needed for regulatory compliance. D3S implements the jurisdiction-specific requirements such as data formats, cryptographic seals, and communication protocols with the regulator.

This guide describes the communication API between the gaming application and D3S.

It contains 3 chapters:

- The present introduction chapter
- Chapter 2 describes the general principles of use of the Web services
- Chapter 3 point to additional information specific to the Web services for Bulgaria

### 1.1 Special note for Bulgaria

The Bulgarian gaming regulator, the **State Commission on Gambling** (subsequently referred to as **SCG**) requires licensed operators to store a number of transactions in a server located in Bulgaria, the **Local Control Server (LCS)**. The data must be available to the SCG, the **National Revenue Agency (NRA)**, and the **State Agency for National Security (SANS)**.

In addition to being a gaming information storage (safe), the LCS must push the data in near real time to the NRA servers via the leased line, and securely store logs on communication, and responses from the servers. The law requires that all the communications between the gaming operator and the NRA be carried out via a leased line connecting the LCS and the NRA IT facilities.

The SCG, NRA, and SANS may at any given time submit a REST/XML query for currently active games to the operator, through the leased line and the LCS. This request must be acknowledged through the same channel.

Dictao’s D3S software and servers provide an architecture which complies with all the regulator’s requirement for the LCS.

Gaming  
application  
D3S Regulator

## 2 Principles

### 2.1 Architecture

This documentation describes the WSDL
1
bindings that will be used to store gaming events for regulatory purposes.

This API is provided in a “Software as a Service” offer. It requires only that the gaming operator fulfills the security requirements (connection, privacy …) and provides correct information about players, gaming status and other notification at the right time.

The safe services will then format, sign, and store corresponding events XML files according to the regulatory rules.

The tracing methods are generic for all countries. But to fulfill regulatory requirements, per-country information are specified inside methods parameters, described in XML Schema data structure.

### 2.2 Environments

Three platforms are available:

- Integration platform: updated to latest version, this service will be maintained to enable adoption and debugging to new customer or during major updates;
- Load testing platform: we may open our load testing platform for customers which are looking for a production configuration dedicated to load balancing tests; load testing schedules must be agreed upon prior to the actual tests
- Production platform: backup, monitored, redundant, load-balanced, this platform is dedicated to our customers and real production environment. It must not be used for either functional or load tests, as records on this platform are published to the regulator

Please be aware that any record sent to the production platform will be handled as a real gaming event, and will be pushed to the NRA servers. Make sure to never send test data in production.

### 2.3 Web Service connection

The applications calling the Web services are authenticated at the D3S by X509 certificate (SSL with mutual authentication) provided by Dictao.

The access control mechanisms of the D3S safe determine if the actor declared at the origin of the action (caller) is really the authenticated actor (by its certificate SSL of authentication).

To specify the certificates on a client program written in Java, such as the samples provided with the integration kit, you will need to specify the following JVM parameters (replace curly brackets with appropriate values):

- -Djavax.net.ssl.keyStore={path/to/your/certificate/yourCertificate.p12}
- -Djavax.net.ssl.keyStorePassword={passwordOfYourKeystore}
- -Djavax.net.ssl.keyStoreType=PKCS12

You can test your connection through the connect method of the “d3sc” command line tool that is available inside the SDK (See the README.txt file in the root directory of the provided archive).

1 http://en.wikipedia.org/wiki/Web_Services_Description_Language

### 2.4 WSDL convention

#### 2.4.1 WSDL definition

The WSDL documents follow the given convention:

- filename: d3s-jel-{PortName}-v1.wsdl
- namespace: http://dictao.com/d3s/jel/bg/service/{PortName}/v{V}.wsdl
- location: https://d3s.dictao.com/d3s/jel/bg/service/{PortName}/v{V}[u{X}]

The WSDL PortName can take one of the following values:

- licensing
- player
- gaming
- echo.

##### 2.4.1.1 Safe writes: asynchronous calls

The WSDL for the ports defines a port type with the following naming convention and methods:

**{PortName}Port**

writeRecords(List<Record> records): String

(echo is a specific case, and does not have a writeRecords method, but an echo method)

The port definition also defines the Record element, which is defined as a choice of different business elements. This way, it is possible to batch in a single web service call, records of various types pertaining to the same domain.

**Record**

Choice

- {BusinessElement1}
- {BusinessElement2}
- ...

The business types are defined in an XSD (see web services reference guide).

All web services calls on writeRecords return a unique string id for the call (**safeCallId**), assigned by the D3S server, and allowing us to trace the call in the server logs.

The call is asynchronous in that, once all the records are transformed into the regulator xml format, and validated against the official regulator xsd, they are safely queued up for further processing and the call ends returning the **safeCallId**.

When a record is later pushed to the NRA server, and if the NRA rejects the data for specific reasons, an email will be sent once a day to an email address that you will have provided to Dictao at registration, with the **safeCallId**, **recordId**, the error code and the error message provided by the NRA server. The NRA expects you to fix the data and send it as soon as possible.

The email will only be sent if there are errors.

##### 2.4.1.2 Synchronous calls (specific to Bulgaria)

The WSDL for the licensing and player ports define, in addition to the writeRecords method described above, a synchronous method use to register new entities with the NRA and obtain an official NRA Id:

**{PortName}Port**

register{Entity}({Entity}RegistrationRequest request): RegistrationResponse

(For the licensing port, {Entity} is Organization, and for the player port {Entity} is Player)

The registerOrganization and registerPlayer method are synchronously calling the NRA server through the LCS. The NRA returns a very important id in the **RegistrationResponse**, which will be used in the subsequent calls to the safe.)

The **RegistrationResponse** contains two values:

- As for writeRecords, a unique string id for the call (**safeCallId**), assigned by the D3S server, and allowing us to trace the call in the server logs.
- A registration id of the entity that was registered, generated by the NRA.

#### 2.4.2 WSDL example

Example of port definition for the gaming service:

filename : d3s-jel-gaming-v1.wsdl  
namespace: https://d3s.dictao.com/d3s/jel/bg/service/gaming/v1.wsdl  
location : https://d3s.dictao.com/d3s/jel/bg/service/gaming/v1

**GamingPort**

writeRecords(List<Record>)

And the corresponding definition of a record:

**Record**

Choice

- gameRecord
- ticketRecord
- activeGamesRecord

### 2.5 XSD convention

#### 2.5.1 XSD definitions

The XSD documents corresponding to the ports follow the given convention:

- filename: d3s-jel-{PortName}-v{YYYY_MM}.xsd
- namespace: https://dictao.com/d3s/jel/bg/service/{PortName}/v{YYYY_MM}.xsd

There are 2 schemas which do not correspond to a single web service port:

- The common schema, d3s-jel-common-v{YYYY_MM}.xsd, contains various type definitions, used in the other xsds.
- The error schema, d3s-jel-error-v1.xsd, contains the type definitions for the exceptions returned by the web services.

A web service port xsd defines the various business elements referenced in the wsdl’s method definition. By convention, all business elements are the xml elements defined in the xsd.

##### 2.5.1.1 Safe writes: asynchronous calls

The elements used in a writeRecords call all extend a base type, **RecordBasicInfo**, which defines the common set of fields.

All operations except for the two register{Entity} operations are asynchronous calls.

**RecordBasicInfo** is defined in the common schema, d3s-jel-common-v{YYYY_MM}.xsd. Here is the definition from d3s-jel-common-v2014_08.xsd:

filename : d3s-jel-common-v2014_08.xsd  
namespace: https://dictao.com/d3s/jel/bg/common/v2014_08.xsd

**RecordBasicInfo**

sequence

- recordId
- GAMonId
- messageDateTime

The **GAMonId** is the operator Id provided by the NRA upon registration using the licensing port’s registerOrganization method.

The **recordId** is a string that the operator provides, that will be used to trace the record throughout its progression through the safe architecture, up until it is sent to the NRA server. If the NRA rejects the records, this **recordId** will be reported back to the operator with the error code and message provided by the NRA servers. It must be unique. Dictao recommends using a UUID for this field (not a counter or any other computed value), possibly with a prefix to distinguish it from various sources / providers.

The **messageDateTime** is the date and time of the record extraction from your information system. We recommend that you provide the time zone offset. If you do not specify the time zone offset, this field has to use Bulgarian time (beware of daylight savings time).

##### 2.5.1.2 Synchronous calls (specific to Bulgaria)

The register{Entity} operations, do not require a **recordId** field, as the official NRA response will be provided synchronously.

Only the two register{Entity} operations are synchronous, all other operations are handled asynchronously.

However, both require a **messageDateTime**. It is the date and time of the registration request as initiated by your information system. We recommend that you provide the time zone offset. If you do not specify the time zone offset, this field has to use Bulgarian time (be careful with daylight savings time).

The remainder of the information is specific to each request.

#### 2.5.2 XSD example

Here is an example of definitions found in the XSD corresponding to the gaming port:

filename : d3s-jel-gaming-v2014_08.xsd  
namespace: https://dictao.com/d3s/jel/bg/gaming/v2014_08.xsd

**gameRecord**  
extension base=RecordBasicInfo

sequence

- gameCode
- gameId
- gameStartTime
- gameEndTime
- numberOfPlayers
- betsAmount
- winningsAmount
- ...

**ticketRecord**  
extension base=RecordBasicInfo

sequence

- gameCode
- ticketDataId
- ticketDataDate
- numberOfTickets
- serialNumberMin
- serialNumberMax
- singleTicketPrice
- totalTicketPrice
- ...

**activeGamesRecord**  
extension base=RecordBasicInfo

sequence

- requestId
- gameCode
- numberOfActiveGames
- activeGameData
- ...

### 2.6 Exception management

All the methods can throw exceptions of two types specific to D3S and exposed by the WSDL:

- **UserFaultException**
- **SystemFaultException**

#### 2.6.1 UserFaultException

They are related to input data provided by the client. They contain an error code specified by the WSDL (file d3s-jel-error-v1.xsd in folder /wsdl/bg/v1).

These exceptions corresponding to problems related to user data, the client application should catch them as it should know how to handle them (modify input data; inform the end user, etc). Choice should be based on the interpretation of the exception code. In any case the client application must not resend this data without correcting it first, or correcting the connection (especially in case of missing privilege, invalid certificate…).

A **safeCallId** is returned as part of the structure, which the client application should log in case you need Dictao to investigate if the safe has more information about the error than what the client application was able to capture.

The D3S server remains operational; it just needs correct input data to provide its services.

Example: invalid input data, like a violation of unique constraint

#### 2.6.2 SystemFaultException

They are related to the execution environment of the D3S.

The client application should log the error and the **safeCallId**. The **safeCallId** will be required if and when you need Dictao to investigate this specific error instance.

These exceptions corresponding to problems on the environment, such as network connectivity, the client application should try again later. A recommended approach, in order to flood a busy safe servers with numerous retries, is to use the “exponential backoff” algorithm, which gradually delays the retries.

Return to a normal service of D3S is usually very quick, thanks to the safe’s high availability architecture. If the problems lies with a single instance, our load balancer will redirect the future calls to the more available instance. If the problems lies in network connectivity problems, the issue will usually disappear in a short time frame.

### 2.7 Records passed as a list in all methods

The method writeRecords takes a list of records as input. This allows processing multiple orders in a single network round trip.

The maximum number of instances in a single call is limited to 1000 or 2Mbytes of data, whichever comes first.

### 2.8 Asynchronous calls return value

When a web service call to the method writeRecords succeeds, that is when no exception is thrown during the call, the server returns a single String value. This value represents an id assigned by the server to the call (since a call may store several records at once in the safe, this **safeCallId** may concern several records and may not be considered as a **recordId**).

This id represents the acknowledgement that the server received the data, and that it takes the responsibility of processing it and making it available to the regulator according to their specifications.

When the web service call fails, through one of the exceptions listed above, a **safeCallId** is also provided as part of the exception fields.

The client application should always log it for reference, as it may be used to track some information in the safe’s server logs.

### 2.9 Idempotence

#### 2.9.1 Principles

The server implements a mechanism to avoid processing multiple times the same records sent repeatedly though asynchronous calls (to the writeRecords method), thus ensuring the web service idempotence (http://en.wikipedia.org/wiki/Idempotence).

Therefore, in any case you have no way to know that your call was received by the D3S server, you should carry out the same call again until you get a response (either an Exception, or a return value).

The idempotence function has the following characteristics:

- There is one history cache per operator.
- The cache keeps record references for one month. Therefore a record sent a second time more than one month after the original record was sent will not be detected and will be processed by the safe.
- A record is identified by all its content, as defined by the regulator. It will therefore include the **messageDateTime** but not the **recordId**. A record sent with a different **messageDateTime** will be processed a second time.

Duplicate calls prevention is very important; this is why we ask you to follow these guidelines to the letter to ensure duplicates are not created.

#### 2.9.2 Guidelines

Only by following the guidelines below can we make sure there are no duplicates in the safe.

- If you want to retry a call, don’t delay too much before retry it; after one month, it will be too late.  
  If you retry this call after 1 month, the records of the original call will be out of the cache and the retry will not be caught as a repetition.
- If you want to retry a call, retry it exactly as it was; do not modify anything in the records, and in particular, do not modify the **messageDateTime**.  
  If you change anything in a record (except **recordId**), the safe will flag it as a different record.

## 3 Additional Documentation

A reference guide is available with the development kit. It describes in details the various elements and types used in the web services.

The development kit itself is a good place to start in order to get familiar with the web services. It contains examples of how to use the different web services calls, and demonstrates with simple use cases the type of data expected for each calls.

A use case documentation is available on Dictao’s Bulgaria web site for partners (https://partenaires.dictao.com/online-gaming-bulgaria).

If you have any questions about the use cases you can send your questions at gaming-compliance-bg@dictao.com. For questions about the web services and how to integrate with our server you can send your questions at gaming-integration@dictao.com. We will make sure to respond promptly.