ansible-role-zabbix-s3-monitoring
=========

Installs and configures scripts which allows Zabbix to automatically discover and monitor file size and timestamps from AWS S3. 

Requirements
------------

Python 3.6+ 
* Boto3 
* JSON

Role Variables
--------------


`s3monitor_aws_access_key`
* AWS IAM Key with at least Read/List permissions for monitored bucket

`s3monitor_aws_secret_access_key` 
* AWS IAM Secret Key with at least Read/List permissions for monitored bucket

`s3monitor_aws_object_prefix`  
* Optional prefix to filter items within a bucket, if not provided all items in bucket will be monitored

`s3monitor_aws_bucket`    
* Bucket containing items to monitor

_Both s3monitor_template_key and s3monitor_template_macro are combined to create the Zabbix key that the Zabbix Agent uses_ 

`s3monitor_template_key`    
* Key base as defined in Application Template in Zabbix (See Requirements)

`s3monitor_template_macro`
* Zabbix Template Key Macro

`s3_key_args` 
 * _Optional_
 * Arguments to append to the filename returned from S3, useful if you need to trim/parse extensions off the file name or get portion of returned path
    

Dependencies
------------

The following need defined in a Zabbix Template to actually use the scripts
* Discovery Rule
    Key should have a `.discovery[]` suffix
 * Size Item Prototype
    Key should have a `.size[]` suffix
 * Timestamp Item Prototype
    Key should have a `.timestamp[]` suffix


Example Playbook
----------------

Demonstration of created files utilizing defined vars

```
    - hosts: servers
      roles:
         - { role: OULibraries.ansible-role-zabbix-s3-monitoring,
               s3monitor_template_key: s3.file, # Key Base defined in Item Prototype 
               s3monitor_template_macro: {#FILE}, # Macro defined in Item Prototype
               s3_key_args: .split("/")[-1] # Get filename from full filepath
           }
```

_userparameter_s3monitor.conf_
```
    UserParameter=s3.file.discovery[*] 
    UserParameter=s3.file.size[*]
    UserParamter=s3.file.timestamp[*]
```
  
_s3monitordiscovery.py_
```
    ...
        # Python List of LLD Item Prototypes
        # This will result in an item being made the key s3.file[{#FILE}] 
        # Where {#FILE} = item['Key'].split("/")[-1]
        
        entry = {"{#FILE}": str(item['Key'].split("/")[-1])}
        if entry not in discovered:
            discovered.append(entry)
```
License
-------

BSD

Author Information
------------------

Cody Bennett - codybennett@ou.edu
