![CLEVER DATA GIT REPO](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/0-clever-data-github.png "李聪明的数据库")

# 自动为AlwaysOn配置新数据库	
### Automatically Configure New Databases For AlwaysOn
**发布-日期:  2016年09月30日 (评论)**

![##############](images/image0012.png?raw=true "#######")

## Contents

- [中文](#中文)
- [English](#English)
- [SQL Logic](#Logic)
- [Build Info](#Build-Info)
- [Author](#Author)
- [License](#License) 


## 中文
这是为AlwaysOn自动配置新数据库的一种非常简单的方法。有没有更好的办法呢？当然，但这只是让你朝着自动化AlwaysOn环境的方向前进的一种方法。
在这个例子中，我将使用一个简单的Sharepoint数据库服务器。
关于MSFT
这是一个基本的2成员节点群集配置（文件共享节点占多数）。

关于SQL Server：
是SQL Server 2014 Enterprise

让配置正常工作你需要些什么？

有一个同时具有操作系统和SQL权限的SQL Server代理服务帐户。
每台服务器上分别有一个相同名称和路径的文件夹。
两个具有相同名称，步骤和逻辑的作业。

关于工作步骤全都是SQL语句。
每台服务器上的文件夹名称：E：\ SQLBACKUPS \ NEW_ALWAYSON_DATABASES
每台服务器上的代理作业：ALWAYSON – CONFIGURE NEW DATABASES
文件夹是此过程中最重要的部分，总体而言，这在很大程度上取决于扩展存储过程：xp_dirtree
我将继续说明每个工作步骤的逻辑（只有5个工作步骤），我们稍后会讨论。目前，这是关于它是如何工作的基本想法。
*检查AlwaysOn配置中涉及的服务器。
*检查可见性组。
*检查当前未配置AlwaysOn的数据库。
*将这些数据库更改为完全恢复。
*禁用完整数据库备份作业。
*禁用事务日志备份作业。
*在本地运行1个完整备份到E：\ SQLBACKUPS \作为'First_Backup_MyDatabase'。
*删除其他服务器文件夹路径上目标中的所有备份文件：％MyOtherServerName％\ E $ \ SQLBACKUPS \ NEW_ALWAYSON_DATABASES
*通过网络对另一个成员服务器进行非完整备份。
*在网络上运行1个事务日志备份到其他成员服务器
*检查本地文件夹（E：\ SQLBACKUPS \ NEW_ALWAYSON_DATABASES）并查看是否在其中找到任何数据库备份文件。
*如果备份文件不存在，在此步骤停止作业（步骤2）。这在日志中表示为“取消作业”，而不是“作业失败”。这就像右键单击并停止作业一样。
*如果存在备份文件，则使用xp_dirtree构建这些备份文件的列表。顺便说一下，备份文件是从另一台服务器上的同一个Job发送到本地文件夹。
*连接到副本服务器，可用性组等。
*将数据库添加到可用性组
*重新启用完全备份作业。
*重新启用事务日志备份作业。
*从本地文件夹E：\ SQLBACKUPS \ NEW_ALWAYSON_DATABASES中删除备份文件
完成。
不要忘记为完整数据库和事务日志备份重新启用作业。
所以基本上，两个作业做的事情完全一样。检查新数据库，备份到另一台服务器，并在那里恢复它们（以及其他各种其他配置）。你可以将这些作业设置为每小时左右运行一次，或者仅在周末运行。顺便说一句，你可能会注意到逻辑上的一些特殊性，你可在需要的地方更改它。另外，我定期加入waitfor延迟。这就是我在每个进程之间给出几秒钟的间隔。例如，当它删除NEW_ALWAYS_ON文件夹中的文件时，我给它大约20秒，以防有相当多的文件在那里，其中一些文件可能很大，可能需要一些时间才能将它们全部删除。 
这是工作步骤：
这是每一步的SQL逻辑：
步骤1：设置数据库属性执行完全备份



## English
Here is an incredibly simple way to automatically configure new databases for AlwaysOn. Is there a better way? Of course; but this is just one method to get you going in the direction of automating your AlwaysOn environment.
In this example; I’ll be using a simple Sharepoint Database Server.
About the MSFT
It’s a basic 2 Member Node Cluster configuration (with file share node majority)
About SQL Server:
It’s SQL Server 2014 Enterprise
What do you need for this configuration to work?
An SQL Server Agent Service Account that has OS, and SQL Rights on both Servers
2 Folders with the same name and path on each server.
2 Jobs with the same name, steps and logic.
About the Job Steps… All pure SQL.
Folder Names on each server: E:\SQLBACKUPS\NEW_ALWAYSON_DATABASES
 on each server: ALWAYSON – CONFIGURE NEW DATABASES
The folders are THE most important part of this process, and overall this depends heavily on the extended stored procedure: xp_dirtree
I’ll go ahead, and layout the logic of each Job Step ( There are only 5 Job steps ) and we’ll get to that later. For now; here’s the basic idea of how this works.
* Checks to see servers involved in AlwaysOn configuration.
* Checks to see Availability Group
* Checks to see what databases are not presently configured for AlwaysOn.
* Changes those databases to Full Recovery.
* Disables Full Database backup job.
* Disables Transaction Log backup job.
* Runs 1 full backup locally to E:\SQLBACKUPS\ as the ‘First_Backup_MyDatabase’.
* Deletes any backup files in the destination on the other servers folder path: %MyOtherServerName%\E$\SQLBACKUPS\NEW_ALWAYSON_DATABASES
* Runs 1 full backup across the network to the other member server.
* Runs 1 transaction log backup across the network to other member server.
* Checks local folder (E:\SQLBACKUPS\NEW_ALWAYSON_DATABASES) to see if any database backup files are found in it.
* If backup files DO NOT exist. ( stop the job at this step (Step 2). This is represented in the log as a ‘Job Cancellation’, not a ‘Job Failure’. It’s just like right-clicking and stopping a job.
* If Backup files exist it builds a list of those backup files using xp_dirtree. Fyi; Backup files were sent to the local folder from the very same Job on the other server.
* Connects to Replica, Availability Groups, etc.
* Adds databases to Availability Group
* Re-Enables Full Backup Job.
* Re-Enables Transaction Log Backup Job.
* Drops the backup files from the local folder E:\SQLBACKUPS\NEW_ALWAYSON_DATABASES
That’s it. 
Don’t forget to re-enable your jobs for full database, and transaction log backups.
So yeah… Basically; both Jobs do exactly the same thing. Checks for new databases, makes a backup across to another server, and restores them there (along with various other configs naturally). You can set these Jobs to run every hour or so, or only on the weekends; what have you. By the way… You may notice some peculiarities with the logic. Just change it where needed. Also; I incorporated waitfor delay periodically. That’s just what I do to give some seconds between each process. For example; when it’s deleting files in the NEW_ALWAYS_ON folder, I give it about 20 seconds in case there are quite a bit a files in there, and some of which might be substantial in size; it might take a few seconds to remove them all.
Here’s the Job Steps:
Here’s the SQL Logic from each step.
STEP 1: Set Database Properties Perform Full Backups


---
## Logic
```SQL
use master; set nocount on
 
-- create temporary table to hold new databases if 
object_id('tempdb..#pending_alwayson_config') is not null
如果object_id（'tempdb ..＃pending_alwayson_config'）不为null，则创建临时表以保存新数据库。
drop    table   #pending_alwayson_config
create  table   #pending_alwayson_config
(
    [server_name]   varchar(255)
,   [database_name] varchar(255)
)
 
-- populate temporary table with list of databases which are not added to AlwaysOn configuration insert into #pending_alwayson_config
使用未添加到AlwaysOn配置插入到#pending_alwayson_config的数据库列表填充临时表。
    select
        'server_name'   = upper(@@servername)
    ,   'database_name' = upper(sd.name)
    from
         master.sys.databases sd
         left join master.sys.dm_hadr_database_replica_cluster_states sdhdrcs on sd.name = sdhdrcs.database_name
    where
         sd.name not in (select database_name from master.sys.dm_hadr_database_replica_cluster_states)
         and sd.database_id > 4
         and sd.state_desc = 'online'
    order by
        sd.name asc
 
declare @availability_group         varchar(255)
declare @other_member_server        varchar(255) 
declare @remove_former_backups      varchar(500)
declare @change_recovery            varchar(max)
declare @set_ao_configs_for_db      varchar(max)
set @availability_group         = (select name from sys.availability_groups)
set @other_member_server        = (select member_name from sys.dm_hadr_cluster_members  where member_type_desc = 'cluster_node' and [member_name] not in (@@servername))
set @remove_former_backups      = 'exec xp_cmdshell ''del \\' + @other_member_server + '\E$\SQLBACKUPS\NEW_ALWAYSON_DATABASES\*.bak'''
set @change_recovery            = ''
set @set_ao_configs_for_db      = ''
 
select  @change_recovery            = @change_recovery +
'alter database [' + name + '] set recovery full;' + char(10)
from    #pending_alwayson_config pac join sys.databases sd on pac.database_name = sd.name
where   sd.recovery_model_desc      = 'simple'
 
select  @set_ao_configs_for_db      = @set_ao_configs_for_db +
'backup database ['             + [database_name] +     '] to disk = ''E:\SQLBACKUPS\First_Backup_' + upper([database_name]) + '.bak'' with format, compression;' + char(10) +
'alter availability group ['            + @availability_group +     '] add database [' + [database_name] + '];' + char(10) +
'backup database ['             + [database_name] +     '] to disk = ''\\' + @other_member_server + '\E$\SQLBACKUPS\NEW_ALWAYSON_DATABASES\' + upper([database_name]) + '.bak'' with format;' + char(10) +
'waitfor delay ''00:00:03'';'           + char(10) +     
'backup log ['                  + [database_name] +     '] to disk = ''\\' + @other_member_server + '\E$\SQLBACKUPS\NEW_ALWAYSON_DATABASES\' + upper([database_name]) + '.trn'';' + char(10)
from    #pending_alwayson_config
 
-- 1. disable full and transaction log backup jobs (yours might be different) change the backup job names accordingly
1.禁用完整和事务日志备份作业（你的可能不同）相应地更改备份作业名称。
exec    msdb..sp_update_job @job_name = 'DATABASE BACKUP LOG  - All Databases', @enabled = 0;
exec    msdb..sp_update_job @job_name = 'DATABASE BACKUP FULL - All Databases', @enabled = 0;
 
-- 2. delete any existing backup files if they exist at the destination server
2.删除目标服务器上存在的任何现有备份文件。
exec    (@remove_former_backups)
 
-- 3. give extra 20 seconds time for files to be deleted at destination server
3.为目标服务器上删除的文件提供额外的20秒时间。
waitfor     delay '00:00:20'
 
-- 4. if any databases are found to have simple recovery model; change to 'full'
4.如果发现任何数据库具有简单的恢复模型，将其改为“full”
exec    (@change_recovery)
 
-- 5. create backup files to local location as 'First_Backup_*', then add to availability group, then backup databases to destination server as MyDatabaseName.bak/.trn
5.将备份文件“First_Backup_*”创建在本地位置，然后添加到可用性组，然后将数据库备份到目标服务器，作为MyDatabaseName.bak / .trn

exec    (@set_ao_configs_for_db)

```

### 第2步：在本地恢复数据库
### STEP 2: Restore Databases Locally

```SQL
-- Restores databases locally if the folder (E:\SQLBACKUPS\NEW_ALWAYSON_DATABASES\) has backup files in it ( sent from other member server ).  If no files exist; step will not perform any actions.
--如果文件夹（E：\ SQLBACKUPS \ NEW_ALWAYSON_DATABASES \）中包含备份文件（从其他成员服务器发送），则在本地还原数据库。如果没有文件，步骤不会执行任何操作。

 
use master;
set nocount on
-- declare path name to where backups reside.
--将路径名声明到备份所在的位置。

declare @path   varchar(255)
set @path   = 'E:\SQLBACKUPS\NEW_ALWAYSON_DATABASES\'
 
-- create temp table to list all backup files within the backup path if object_id('tempdb..#dirtree') is not null
--如果object_id（'tempdb ..＃dirtree'）不为null，则创建临时表以列出备份路径中的所有备份文件
      drop table #dirtree;
 
create table #dirtree
(
    id      int identity(1,1)
,   subdirectory    nvarchar(512)
,   depth       int
,   isfile      bit
);
 
-- populate temp table with backup file list insert into #dirtree (subdirectory, depth, isfile) exec master..xp_dirtree @path, 1, 1
--用备份文件列表填充临时表插入到# dirtree (subdirectory, depth, isfile) exec master..xp_dirtree @path, 1, 1。


-- restore the databases with no recovery - restore full backups first, then transaction logs second if not exists(select 1 from #dirtree)
--从备份恢复数据库 - 首先恢复完整备份，然后再次恢复事务日志（从#dirtree中选择1）

    begin
        print 'There are no new databases to add to the AlwaysOn configuration.'
        -- enable database full and log backup jobs
        exec    msdb..sp_update_job @job_name = 'DATABASE BACKUP LOG  - All Databases', @enabled = 1;
        exec    msdb..sp_update_job @job_name = 'DATABASE BACKUP FULL - All Databases', @enabled = 1;
        -- stop job at this step
        exec msdb..sp_stop_job 'ALWAYSON - CONFIGURE NEW DATABASES'
    end
    else
        begin
        print 'New Databases exist so they will be added to the AlwaysOn configuration.'
        declare @restore    varchar(max)
        set @restore    = ''
        select  @restore    = @restore + 
            case 
                when right(subdirectory, 3) = 'bak' then 'restore database ['   + replace(subdirectory, '.bak', '') +       '] from disk = ''' + @path + subdirectory + ''' with norecovery;' + char(10)
                when right(subdirectory, 3) = 'trn' then 'restore log ['        + replace(subdirectory, '.trn', '') +       '] from disk = ''' + @path + subdirectory + ''' with norecovery;' + char(10)
            end
        from    #dirtree where subdirectory like '%.bak' or subdirectory like '%.trn' order by id asc
        exec (@restore)
    end

```

### STEP 3: Connect To Replicas


```SQL
use master;
set nocount on
 
declare @availability_group varchar(255)
declare @connect_to_replica varchar(max)
set @availability_group = (select name from sys.availability_groups)
set @connect_to_replica =
'
-- Wait for the replica to start communicating begin try declare @conn bit declare @count int declare @replica_id uniqueidentifier declare @group_id uniqueidentifier set @conn = 0 set @count = 30 -- wait for 5 minutes
--等到副本开始通信时开始尝试声明@conn位声明@count int声明@replica_id uniqueidentifier声明@group_id uniqueidentifier set @conn = 0 set @count = 30 --等待5分钟
 
 
if (serverproperty(''IsHadrEnabled'') = 1)
    and (isnull((
    select member_state 
    from master.sys.dm_hadr_cluster_members 
    where upper(member_name COLLATE Latin1_General_CI_AS) = upper(cast(serverproperty(''ComputerNamePhysicalNetBIOS'') as nvarchar(256)) COLLATE Latin1_General_CI_AS)), 0) != 0)
    and (isnull((select state from master.sys.database_mirroring_endpoints), 1) = 0) begin
    select @group_id = ags.group_id 
    from    master.sys.availability_groups as ags 
    where   name = N''' + @availability_group + '''
    select  @replica_id = replicas.replica_id 
    from    master.sys.availability_replicas as replicas 
    where   upper(replicas.replica_server_name COLLATE Latin1_General_CI_AS) = upper(@@SERVERNAME COLLATE Latin1_General_CI_AS) and group_id = @group_id
    while   @conn != 1 and @count != 0
    begin
        set @conn = isnull((select connected_state from master.sys.dm_hadr_availability_replica_states as states where states.replica_id = @replica_id), 1)
        if @conn = 1
        begin
            -- exit loop when the replica is connected, or if the query cannot find the replica status
            break
        end
        waitfor delay ''00:00:10''
        set @count = @count - 1
    end
end
end try
begin catch
end catch '
-- If the wait loop fails, do not stop execution of the alter database statement 

--如果wiat循环异常，不需要停止执行改变数据库的语句。
 
exec (@connect_to_replica)

```

### STEP 4: Add Databases To Availability Groups 

```SQL
use master; set nocount on
-- declare path name to where backup files reside.
--第4步：将数据库添加到可用性组。
declare @path   varchar(255)
set @path   = 'E:\SQLBACKUPS\NEW_ALWAYSON_DATABASES\'
 
-- create temp table to list all backup files within the backup path if object_id('tempdb..#dirtree') is not null
--如果object_id（'tempdb ..＃dirtree'）不为null，则创建临时表以列出备份路径中的所有备份文件。

      drop table #dirtree;
 
create table #dirtree
(
    id      int identity(1,1)
,   [subdirectory]  nvarchar(512)
,   depth       int
,   isfile      bit
);
 
-- populate temp table with backup file list insert into #dirtree ([subdirectory], depth, isfile) exec master..xp_dirtree @path, 1, 1
--用备份文件列表填充临时表插入到#dirtree（[subdirectory], depth, isfile）exec master..xp_dirtree @path，1,1

 
-- delete *.trn files from file list and drop *.bak extension from file list
--从文件列表中删除* .trn文件，并从文件列表中删除* .bak扩展名。

delete from #dirtree    where   [subdirectory] like '%.trn';
update      #dirtree    set [subdirectory] = replace([subdirectory], '.bak', '')
 
-- produce and execute sql logic to add database to availability group
--生成并执行sql逻辑以将数据库添加到可用性组。

declare @availability_group     varchar(255)
declare @add_database_to_ag varchar(max)
set @availability_group     = (select name from sys.availability_groups)
set @add_database_to_ag = ''
select  @add_database_to_ag = @add_database_to_ag +
'alter database [' + [subdirectory] + '] set hadr availability group = [' + @availability_group + '];' + char(10)
from    #dirtree order by [subdirectory] asc
exec    (@add_database_to_ag)

```

### 步骤5：删除以前的新AlwaysOn数据库备份
### STEP 5: Remove Former New AlwaysOn Database Backups


```SQL

use master; set nocount on
 
-- enable database full and log backup jobs
--启用数据库完整和日志备份作业

exec    msdb..sp_update_job @job_name = 'DATABASE BACKUP LOG  - All Databases', @enabled = 1;
exec    msdb..sp_update_job @job_name = 'DATABASE BACKUP FULL - All Databases', @enabled = 1;
 
-- remove former backup files
--删除以前的备份文件

declare @remove_former_backups  varchar(500)
set @remove_former_backups  = 'exec xp_cmdshell ''del /Q E:\SQLBACKUPS\NEW_ALWAYSON_DATABASES\*.*'''
exec    (@remove_former_backups)

```


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

- **李聪明的数据库 Lee's Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明的数据库** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明的数据库-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/1-clever-data-github.png "李聪明的数据库")

