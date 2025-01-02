## Delta
- originally developed by Databricks and open-sourced
- metadata stored in file system/object store meaning there is no separate database like in Hive
- log files are the single source of truth to tell consumers which files are present in the table.
  Allows for partial writes from failed jobs to not pollute the table. (Unlike Hive where all files
  under a path are assumed part of the table)
- Using a log like this means that operations that modify or delete rows can be
  represented as deleting old files and creating new ones (that contain those rows)
  so the whole partition doesn't need to be re-written
  
