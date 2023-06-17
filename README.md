# FileEncryptionClientServer
This is a client server project. 
The server acts as a file server allowing the user to upload a file to it's storage in an encrypted channel using the Client.
The Server is written in python and the Client is written in C++.
This project is written as part of an assignment for Defensive-System-Programming course at the Open University of Israel.
The software architecture is based on client-server. The client initiates the communication to the server by the exchanging the encryption keys with it, then procceeds to transfer the requested file to the it via encrypted communication. 
The client verifies then the server received the file properly by comparing the checksum on both sides, and if it did not pass properly, tries to transfer again (up to 3 attempts).

## Logic Flow
<img src="Logical Client Server Flow.png" alt="Logical Client Server Flow" height=600>

## Server Data Handling
The server saves the clients' and files' data in the memory (RAM). Additionally, it maintains a SQLite database with the following tables:

### Clients Table
|Column Name|Column Type    |Column Description         |
|-----------|---------------|---------------------------|
|id         |varchar(16)    |Client Unique Identifier   |
|name       |varchar(127)   |Client Name                |
|public_key |varchar(160)   |Client Public Key          |
|last_seen  |datetime       |Last Seen Time of Client   |
|shared_key |varchar(32)    |Shared Key with Server     |

### Files Table
|Column Name|Column Type    |Column Description         |
|-----------|---------------|---------------------------|
|file_name  |varchar(255)   |File Name                  |
|file_path  |varchar(255)   |File Path                  |
|client_id  |integer        |Uploader Unique Identifier |
|verified   |boolean        |Was File Verified by Client|

## Communication Protocol
### Requests
#### Request Header
|Field          |Size (in bytes)|Description                |
|---------------|---------------|---------------------------|
|Client ID      |255 bytes      |File Name                  |
|Version        |byte           |Client Version             |
|Code           |2 bytes        |Request Code               |
|Payload Size   |4 bytes        |Size of Request Payload    |

#### Registration Request (1100)
|Field          |Size (in bytes)|Description                |
|---------------|---------------|---------------------------|
|Name           |255 bytes      |Client name                |

#### Public Key Sending Request (1101)
|Field          |Size (in bytes)|Description                |
|---------------|---------------|---------------------------|
|Name           |255 bytes      |Client name                |
|Public Key     |160 bytes      |Client Public Key          |

#### File Sending Request (1103)
|Field          |Size (in bytes)|Description                |
|---------------|---------------|---------------------------|
|Client ID      |16 bytes       |Client id                  |
|Content Size   |4 bytes        |File Size                  |
|File Name      |255 bytes      |File Full Path             |
|File Content   |variable       |File Content(AES Encrypted)|

#### Valid CRC Request (1104) & Invalid CRC, Resending (1105) & Invalid CRC, Finished (1106)
|Field          |Size (in bytes)|Description                |
|---------------|---------------|---------------------------|
|Client ID      |16 bytes       |Client id                  |
|File Name      |255 bytes      |File Full Path             |

### Responses
#### Response Header
|Field          |Size (in bytes)|Description                |
|---------------|---------------|---------------------------|
|Version        |1 byte         |Client Version             |
|Code           |2 bytes        |Request Code               |
|Payload Size   |4 bytes        |Size of Request Payload    |

#### Registration Response (2100) & Registration Response (2101)
|Field          |Size (in bytes)|Description                |
|---------------|---------------|---------------------------|
|Client ID      |16 bytes       |Client Given ID            |

#### Encrypted AES Key Response (2102)
|Field          |Size (in bytes)|Description                |
|---------------|---------------|---------------------------|
|Client ID      |16 bytes       |Client Given ID            |
|AES Symmetric  |variable       |Client Given ID            |

#### File Received with CRC Response (2103)
|Field          |Size (in bytes)|Description                |
|---------------|---------------|---------------------------|
|Client ID      |16 bytes       |Client id                  |
|Content Size   |4 bytes        |File Size                  |
|File Name      |255 bytes      |File Full Path             |
|Checksum       |4 bytes        |File Content CRR           |

#### Message Received Response (2104)