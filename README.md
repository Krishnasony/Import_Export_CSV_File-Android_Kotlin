# Import_Export_CSV_File-Android_Kotlin
Import and Export .csv file in Room database as a Table(Android-Kotlin)

Firstly, Add Two file in Your Android package  CSVReader and CSVWriter(get this file from this repository)

#Eport CSV file from your room database table
``` 
    private fun exportCSV(){
        val exportDir = File(getExternalStorageDirectory(), "/CSV")// your path where you want save your file
        if (!exportDir.exists()) {
            exportDir.mkdirs()
        }

        val file = File(exportDir, "$TABLE_NAME.csv")//$TABLE_NAME.csv is like user.csv or any name you want to save
        try {
            file.createNewFile()
            val csvWrite = CSVWriter(FileWriter(file))
            val curCSV = database.query("SELECT * FROM $TABLE_NAME", null)// query for get all data of your database table
            csvWrite.writeNext(curCSV.columnNames)
            while (curCSV.moveToNext()) {
                //Which column you want to export
                val arrStr = arrayOfNulls<String>(curCSV.columnCount)
                for (i in 0 until curCSV.columnCount - 1) {
                    when (i) {
                        20, 22 -> {
                        }
                        else -> arrStr[i] = curCSV.getString(i)
                    }
                }
                csvWrite.writeNext(arrStr)
            }
            csvWrite.close()
            curCSV.close()
            showToast("Exported SuccessFully",this)
        } catch (sqlEx: Exception) {
            Timber.e(sqlEx)
        }

    } 
```    
    
  #Import CSV file from your storage path

 ``` 
 private fun importCSV(){
        val csvReader =
            CSVReader(FileReader("${getExternalStorageDirectory()}/CSV/$TABLE_NAME.csv"))/* path of local storage (it should be your csv file locatioin)*/
        var nextLine: Array<String> ? = null
        var count = 0
        val columns = StringBuilder()
        GlobalScope.launch(Dispatchers.IO) {
            do {
                val value = StringBuilder()
                nextLine = csvReader.readNext()
                nextLine?.let {nextLine->
                    for (i in 0 until nextLine.size - 1) {
                        if (count == 0) {                             // the count==0 part only read
                            if (i == nextLine.size - 2) {             //your csv file column name
                                columns.append(nextLine[i])          
                                count =1             
                            }
                            else
                                columns.append(nextLine[i]).append(",")
                        } else {                         // this part is for reading value of each row
                            if (i == nextLine.size - 2) {
                                value.append("'").append(nextLine[i]).append("'")
                                count = 2
                            }
                            else
                                value.append("'").append(nextLine[i]).append("',")
                        }
                    }
                    if (count==2) {
                        customerViewModel.pushCustomerData(columns, value)//write here your code to insert all values
                    }
                }
            }while ((nextLine)!=null)
        }

        showToast("Imported SuccessFully",this)
    }
```    


#first add this fun into your Dao
 ``` @RawQuery
    fun insertDataRawFormat(query: SupportSQLiteQuery): Boolean? 
  ```
#And Add this fun in your viewModel or In Repo and call it like I called  here
/*if (count==2) { customerViewModel.pushCustomerData(columns, value)}*/

```
suspend fun pushCustomerData(columns:StringBuilder,values:StringBuilder) = withContext(Dispatchers.IO){
        val query = SimpleSQLiteQuery(
            "INSERT INTO customer ($columns) values($values)",
            arrayOf()
        )
        customerDao.insertDataRawFormat(query)
    }
 ```
