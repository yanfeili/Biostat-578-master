Biostat-578-master
==================
#BIOST 578 Homework 1

#Question 1: Use the GEOmetabd package to find all HCV gene expression data using the Illumina platform submitted by an investigator at Yale.This should be done with a single query, showing the title, the GSE accession number, the GPL accession number and the manufacturer and the description of the platform used.

source("http://bioconductor.org/biocLite.R")
# To install the required packages GEOmetadb and GEOquery
biocLite("GEOmetadb","GEOquery")

# Open the GEOmetadb library
library(GEOmetadb)

# To download the entire database and connect to this database
getSQLiteFile()
geo_con<-dbConnect(SQLite(),'GEOmetadb.sqlite') 

# To see all the tables in the geo_con databases
dbListTables(geo_con)  
dbListFields(geo_con, 'gsm')  # TO see the variables in gsm table
dbListFields(geo_con, 'gse')  # To see the variables in gse table
dbListFields(geo_con, 'gpl')  # To see the variables in gpl table


# For specific variable, you could also find as below.
res<-dbGetQuery(geo_con,"SELECT gse.title FROM gse")
head(res)

# To find all HCV gene expression data using the Illumina platform submitted. 
res <-dbGetQuery(geo_con, "SELECT gse.title, gse.gse, gpl.gpl, gpl.manufacturer,gpl.description FROM (gse JOIN gse_gpl ON gse.gse=gse_gpl.gse)j JOIN gpl ON j.gpl=gpl.gpl WHERE gse.title LIKE '%HCV%' AND gpl.title like'%Illumina%' AND gse.contact LIKE '%Yale%'")
res

# Question 2:  The same output can also be achieved through the R package data.table.
library(data.table)

# Next we need to convert the three files from GEOmetadb package into data.table tables:

dt_gse<-data.table(dbReadTable(geo_con,'gse'))
dt_gpl<-data.table(dbReadTable(geo_con,'gpl'))
dt_gse_gpl<- data.table(dbReadTable(geo_con,'gse_gpl'))

merge( merge( (dt_gse[like(contact,"Yale")][like(title, "HCV")]), dt_gse_gpl, by = "gse",  suffixes = c(".gse_inner",".gse_gpl") ), dt_gpl[like(title, "Illumina")], by = "gpl", suffixes = c(".gse", ".gpl") )[,list(title.gse, gse, gpl, manufacturer, description)]
