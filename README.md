![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")        

# Use SQL To Restore All SQL Databases Using Only A Backup Folder
**Post Date: April 10, 2014**        



## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process

<p>Here's some logic I wrote to take get the database names for your entire server, and pass that into some restore logic which in turns produces a restore script for every database you have on the server. ( Excluding the System Databases Master, Model, Msdb, Tempdb )
All you need to do is make sure you set your path for the @backup_file_name, and run it on your Database Server, and then click on the XML output under the results, and paste it in your destination server. Assuming your destination database server has access to the share it will restore every database you have on it. Be sure to change the /* and */ from the 'exec' near the bottom. Just do a find and replace after you paste it. Find /*exec*/ and replace with exec. Thats it. Enjoy.
Note: later on I might include some additional logic that will get a list of all backup files using the xp_dirtree. Here's a quick example:</p>     


## SQL-Logic
```SQL
use master;
exec xp_dirtree '\\MyShare\',1,1
This will basically show you a list of all backup files which you can use to pass into an @table, and then in-turn pass that into the logic below. Some extra coding involved, but makes for a great, and easy way to run restores based on nothing more than a path to where your backup files. Should help simplify your migrations. I've been using this method for some time so I know it will work for you.
use master;
set nocount on
declare @create_restore_logic varchar(max) set @create_restore_logic = "
select @create_restore_logic = @create_restore_logic + '
use master;
set nocount on
go
declare @database_name varchar (255) declare @backup_file_name varchar (255) set @database_name = "' + replace(name, "", """) + "'
set @backup_file_name = "\\MyShare\' + replace(name, "", ") + '.bak" declare @filelistonly table
(
logicalname nvarchar (128) , physicalname nvarchar (260) , [type] char (1)
, filegroupname nvarchar (128) , size numeric (20,0) , maxsize numeric (20,0) , fileid bigint
, createlsn numeric (25,0) , droplsn numeric (25,0) , uniqueid uniqueidentifier
, readonlylsn numeric (25,0) , readwritelsn numeric (25,0) , backupsizeinbytes bigint
, sourceblocksize int
, filegroupid int
, loggroupguid uniqueidentifier
, differentialbaselsn numeric (25,0) , differentialbaseguid uniqueidentifier
, isreadonl bit
, ispresent bit
, tdethumbprint varbinary (32)
)
insert into
@filelistonly exec ("restore filelistonly from disk = """ + @backup_file_name + """") declare @restore_line0 varchar (255) declare @restore_line1 varchar (255) declare @restore_line2 varchar (255) declare @stats varchar (255) declare @move_files varchar (max) set @restore_line0 = ("use master; ")
set @restore_line1 = ("exec master..sp_killallprocessindb """ + @database_name + """;")
set @restore_line2 = (select "restore database [" + @database_name + "] from disk = """ + @backup_file_name + """ with replace, recovery, ") set @stats = ("stats = 20;")
set @move_files = ""
select @move_files = @move_files + "move """ + logicalname + """ to """ + physicalname + """," + char(10) from @filelistonly order by fileid asc
/*exec*/
(
@restore_line0
+ @restore_line1
+ @restore_line2
+ @move_files
+ @stats
)
go
'
from sys.databases where name not in ('master', 'model', 'tempdb', 'msdb') order by name asc
select (@create_restore_logic) for xml path ("), type

```


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

    
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")

