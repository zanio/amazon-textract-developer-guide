# Exporting Tables into a CSV File<a name="examples-export-table-csv"></a>

These Scala examples show how to export tables from an image of a document into a comma\-separated values \(CSV\) file\.

The example for synchronous document analysis collects table information from a call to [AnalyzeDocument](API_AnalyzeDocument.md)\. The example for asynchronous document analysis makes a call to [StartDocumentAnalysis](API_StartDocumentAnalysis.md) and then retrives the results from [GetDocumentAnalysis](API_GetDocumentAnalysis.md) as `Block` objects\.

Table information is returned as [Block](API_Block.md) objects from a call to [AnalyzeDocument](API_AnalyzeDocument.md)\. For more information, see [Tables](how-it-works-tables.md)\. The `Block` objects are stored in a map structure that's used to export the table data into a CSV file\. 

------
#### [ Synchronous ]

In this example, you will use the functions: 
+ `get_table_csv_results` – Calls [AnalyzeDocument](API_AnalyzeDocument.md), and builds a map of tables that are detected in the document\. Creates a CSV representation of all detected tables\.
+ `generate_table_csv` – Generates the CSV file for an individual table\.
+ `get_rows_columns_map` – Gets the rows and columns from the map\.
+ `get_text` – Gets the text from a cell\.

**To export tables into a CSV file**

1. Configure your environment\. For more information, see [Prerequisites](examples-blocks.md#examples-prerequisites)\.

   1. Save the following example code to a file named *textract\_python\_table\_parser\.py*\.

 ```scala
import akka.stream.Materializer
import akka.stream.scaladsl.{Sink, Source}
import com.amazonaws.services.textract.model.{Block, DocumentMetadata}
import spray.json.DefaultJsonProtocol

import scala.collection.immutable.ListMap
import scala.collection.mutable.ListBuffer
import scala.concurrent.{ExecutionContext, Future}
import scala.jdk.CollectionConverters.CollectionHasAsScala
object AwsModel {

   case class DocumentLocation(S3ObjectName: String, S3Bucket: String)

   case class DocumentAnalysisResult(blocks: List[Block], documentMetadata: DocumentMetadata, nextToken: Option[String], status: String){

   }


   case class MessageResponse(JobId: String,  API: String, Status: String,Timestamp: Long, JobTag: Option[String], DocumentLocation: DocumentLocation)

   object AwsModelJsonProtocol extends DefaultJsonProtocol {
      implicit val documentLocationFormat = jsonFormat2(DocumentLocation)
      implicit val messageResponseFormat = jsonFormat6(MessageResponse)

   }

   def convertBlockToCsv(blocks: List[Block], messageResponse: MessageResponse)(implicit  ec: ExecutionContext,  mat: Materializer): Future[(ListBuffer[List[String]], MessageResponse)] = {
      val blocksMap = blocks.map(block => block.getId -> block).toMap
      val tables = blocks.filter(_.getBlockType == "TABLE")
      Source(tables).zipWithIndex.fold(ListBuffer.empty[List[String]])((acc, blockTuple) => {
         val emptyVal = ListBuffer(List(""), List(""), List(""))
         val (block: Block, index: Long) = blockTuple
         Option(block.getRelationships) match {
            case Some(_) =>
               val newCsvAcc = generateTableCsv(block, blocksMap, index + 1L)
               acc  ++ newCsvAcc ++ emptyVal
            case None => acc ++ emptyVal
         }

      }).runWith(Sink.head).map((_, messageResponse))
   }
   

   def generateTableCsv(table: Block, blocksMap: Map[String, Block], tableIndex: Long): ListBuffer[List[String]] = {
      val tableId = s"Table_${tableIndex}"
      val rows = getRowsColumnMap(table, blocksMap)
      var csv = s"Table: ${tableId}\n\n"
      val listMap = ListMap(rows.toSeq.sortBy(_._1): _*)
      val rowList = ListBuffer.empty[List[String]]
      listMap.foreach {
         case (rowIndex, columns) =>
            val newlistMap = ListMap(columns.toSeq.sortBy(_._1): _*)
            if (newlistMap.nonEmpty) {
               val columList = ListBuffer[String]()
               newlistMap.foreach {
                  case (columnIndex, text) =>
                     val newWord = text.replaceAll("\n", " ")
                     columList.append(s"$newWord")
               }
               rowList.append(columList.toList)
            }
      }
      if (rows.nonEmpty) rowList else ListBuffer.empty[List[String]]
   }

   def getRowsColumnMap(tableResult: Block, blocksMap: Map[String, Block]): Map[Integer, Map[Integer, String]] = {
      Option(tableResult.getRelationships) match {
         case Some(value) => value.asScala.toList.filter(_.getType == "CHILD")
                 .map(_.getIds.asScala.toList).flatMap(_.toList)
                 .filter(e => blocksMap.contains(e) && blocksMap(e).getBlockType == "CELL")
                 .foldLeft(Map.empty[Integer, Map[Integer, String]])((acc, e) => {
                    val cell = blocksMap(e)
                    val rowIndex = cell.getRowIndex
                    val columnIndex = cell.getColumnIndex
                    val innerMap = acc.getOrElse(rowIndex, Map.empty[Integer, String])
                    val updatedInnerMap = innerMap + (columnIndex -> getText(cell, blocksMap))
                    acc + (rowIndex -> updatedInnerMap)
                 })
         case None => Map.empty[Integer, Map[Integer, String]]
      }
   }

   def getText(block: Block, blocksMap: Map[String, Block]): String = {
      Option(block.getRelationships) match {
         case Some(relationships) => {
            relationships.asScala.toList.filter(_.getType == "CHILD")
                    .map(_.getIds.asScala.toList).flatMap(_.toList)
                    .filter(blocksMap.contains)
                    .fold("")((acc, e) => {
                       if (blocksMap(e).getBlockType == "WORD") {
                          acc + blocksMap(e).getText + " "
                       } else {
                          acc
                       }
                    })
         }
         case None => ""
      }

   }

}

 ```

1. At the command prompt, enter the following command\. Replace `file` with the name of the document image file that you want to analyze\.

   ```
   python textract_python_table_parser.py file
   ```

When you run the example, the CSV output is saved in a file named `output.csv`\.

------
#### [ Asynchronous ]

In this example, you will use make use of two different scripts\. The first script starts the process of asynchronoulsy analyzing documents with `StartDocumentAnalysis` and gets the `Block` information returned by `GetDocumentAnalysis`\. The second script takes the returned `Block` information for each page, formats the data as a table, and saves the tables to a CSV file\.

**To export tables into a CSV file**

1. Configure your environment\. For more information, see [Prerequisites](examples-blocks.md#examples-prerequisites)\.

1. Ensure that you have followed the instructions given at see [Configuring Amazon Textract for Asynchronous Operations](api-async-roles.md)\. The process documented on that page enables you to send and receive messages about the completion status of asynchronous jobs\.

1. In the following code example, replace the value of `roleArn` with the Arn assigned to the role that you created in Step 2\. Replace the value of `bucket` with the name of the S3 bucket containing your document\. Replace the value of `document` with the name of the document in your S3 bucket\. Replace the value of `region_name` with the name of your bucket's region\.

   Save the following example code to a file named *start\_doc\_analysis\_for\_table\_extraction\.py\.*\.

   

1. Run the code\. The code will print a JobId\. Copy this JobId down\.

1.  Wait for your job to finish processing, and after it has finished, copy the following code to a file named *get\_doc\_analysis\_for\_table\_extraction\.py*\. Replace the value of `jobId` with the Job ID you copied down earlier\. Replace the value of `region_name` with the name of the region associated with your Textract role\. Replace the value of `file_name` with the name you want to give the output CSV\.



1. Run the code\.

   After you have obtained you results, be sure to delete the associated SNS and SQS resources, or else you may accrue charges for them\.

------