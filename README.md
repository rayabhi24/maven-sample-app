# Maven sample app

## Installation

- Install Java
- Install Maven

- Build package: `$ mvn clean install`

- Run APP: `$ mvn tomcat:run`
  

From master branch
From Develop branch
my new awesome change From master branch
new change
new change
new change for github desktop


/************************************************************************************************************************************************
Description :- We have different types of reports which has multiple services running on servers so using this framework we can fire 'n' number of XML requests
and extract actual response for comparison with expected responses and save the result in report format and framework is capable of converting the PDF XML request into PDF format.
Output result is getting saved in the defined places.

Technology : Groovy Scripting, XMLUnit

Pre-requisites :- Data Driven Approach so enrich the data in to files

Created Date :- June Cycle

Version :- 1.0

Authur :- Capgemini QA Team

*************************************************************************************************************************************************/
import com.eviware.soapui.impl.wsdl.teststeps.registry.WsdlTestRequestStepFactory
import org.custommonkey.xmlunit.*

/* To use Microsoft Excelsheet we need below POI lib methods to be enabled.
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.hssf.usermodel.*;
import org.apache.poi.xssf.usermodel.*;
import org.apache.poi.ss.util.*;
import org.apache.poi.ss.usermodel.*;
*/
import java.io.*;
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import javax.xml.bind.DatatypeConverter;
import org.apache.commons.io.FilenameUtils

//GroovyUtils  is an API
import com.eviware.soapui.support.GroovyUtils


XMLUnit.setIgnoreWhitespace(true)
XMLUnit.setIgnoreComments(true)
XMLUnit.setIgnoreDiffBetweenTextAndCDATA(true)
XMLUnit.setNormalizeWhitespace(true)

//Declaring Variables

      def path = "C:/Users/vxs178/Desktop/AutomationTestFrameWork/DataDrivenExcelSheet/SOAPUITESTDATA.txt"
      def FolderPath = "C:/Users/vxs178/Desktop/AutomationTestFrameWork/VedaSOAPUI/"
      

     def noOfColumns
     def noOfRows
     def rowValue
     def CellHeader;
     def CellValue
     def testStepName;
     def headerValue
     def headerValueNamespace
     def runHeader;
     def toRun
      InputStream inputStream = new FileInputStream(path);
      
      def textFile = new File(path)
      def textFileLines = textFile.readLines()
      
     //declaring arraylist for status and steps
     List<String> testCaseStatus = new ArrayList<String>()
     List<String> Report_Type = new ArrayList<String>()
     def status;
     def failCount=0, passCount=0;
                
     for(int i=0;i<textFileLines.size();i = i+3){
            CellValue_Request = textFileLines[i]
            CellValue_Response = textFileLines[i+1]
            CellValue_Endpoint = textFileLines[i+2]
        
            // get the current testCase to add testSteps later
            def tc = testRunner.testCase;
            // get the SOAP TestStep as template to create the other requests
            def tsTemplate = tc.getTestStepByName("RunTimeRequest")

            tsTemplate.getHttpRequest().setEndpoint(CellValue_Endpoint)

            // create the test step factory to use later
            def testStepFactory = new WsdlTestRequestStepFactory();

           // your location (We may need to paramterised the location for different folder structure)

            def directory = new File(FolderPath + CellValue_Request)
            // for each file in the directory
            directory.eachFile{ file -> 
                  // use file name as test step name 
            testStepName = file.getName()
             //log.info "testStepName= "+ testStepName.subString(1,testStepName.length()-4)
                  def curentPath = new File(file.getParent());
                  def curentPath1 = new File(curentPath.getParent());
                 // log.info "looking for " +curentPath1
                  Report_Name= curentPath1.getName()
                  Report_Type.add("\""+Report_Name+"\"")
              //    log.info (Report_Type)
                  
                  // create the config
                  def testStepConfig = testStepFactory.createConfig( tsTemplate.getTestRequest(), testStepName)

                  // add the new testStep to TestCase
                  def newTestStep = tc.insertTestStep( testStepConfig, -1 ) 
                  log.info "NewTestStep  ---> " +testStepName
                  
                  // set the request from the file content
                  newTestStep.getTestRequest().setRequestContent(file.getText())
                  testStepContext = new com.eviware.soapui.impl.wsdl.testcase.WsdlTestRunContext(newTestStep); 
                  newTestStep.run(testRunner, testStepContext); 

                  // Save the server side response accoding to the request file and store in a variable 
                  def ActualResponse = context.expand( '${' + testStepName + '#Response}' )
                  def ActualRunTimeResponseFolderName = FolderPath + CellValue_Response + "/ActualResponse"
                  def ActualRunTimeResponse = new File(ActualRunTimeResponseFolderName); 
                  if(!ActualRunTimeResponse.exists()) {
                        ActualRunTimeResponse.mkdirs();
                  }
                  def myOutFile = FolderPath + CellValue_Response + "/ActualResponse/"+ testStepName
                  print myOutFile
                  def AResponse = context.expand( '${' + testStepName + '#Response}' )
                  def f = new File(myOutFile)
                  f.write(AResponse,"ISO-8859-1")
                 

                  /*
                   def modifiedString="AResponse"
                   def pdf = readPDFTradingHistory(modifiedString)
                  def outputLocation="D:/Vikas/AutomationTestFrameWork/VedaSOAPUI/Response/ActualResponse"
                  encodedPDF(String pdf, String outputLocation)

                  */
                  /*

                  //  class PacketBeatPDFDecoder {
                    String modifiedString = "AResponse";
                    */
                                                                String outputLocation = FolderPath + CellValue_Response + "/PDFActual/"
                                                                //String outputfileName = "DecodedPdf";
                                                                //String pdf;
                                                                
   
                                                xmlHolder  = (new com.eviware.soapui.support.GroovyUtils( context )).getXmlHolder(AResponse)
                                              xmlHolder.namespaces["ns"] = "http://vedaxml.com/vxml2/veda-th-pdf-response-v1.0"
                                                pdfNode = xmlHolder.getNodeValue("//ns:pdf")
                                                               // log.info "pdfNode "+pdfNode
                                                                //XmlHolder xmlHolder = new XmlHolder("AResponse");

                                                             //   log.info "hellooutput" + outputLocation + "\\" + testStepName + ".pdf" 

                         try{

                                 if (pdfNode != null) {
                                File dir = new File(outputLocation);
                                dir.mkdirs();
                                 
                                      // log.info "output" + outputLocation + "\\" + testStepName + ".pdf"  
                                      //PDFFileName =  (testStepName).removeextension 
               
                                     PDFFileName = testStepName.substring(0,testStepName.length()-4)  
                                                                    
                                byte[] decodedPdfByteArray = DatatypeConverter.parseBase64Binary(pdfNode);
                                                               
                                                                                log.info "ActualpdfNode"
                                                                                BufferedOutputStream bos = new BufferedOutputStream(
                                                                                                                new FileOutputStream(outputLocation + "\\" + PDFFileName + ".pdf"));
                                                                                bos.write(decodedPdfByteArray, 0, decodedPdfByteArray.length);
                                                                                bos.flush();
                                                                                bos.close();
                                                                                log.info "Actual PDF Generated and Saved Succeccfully"  
                                                         
                                                                }
                         }
                                catch (Exception e) {
                                log.info "Exception" +e
                                }
                 // }

                  /*static void main(String[] args) {

                   readPDFTradingHistory(modifiedString);
                   encodedPDF(pdf,outputLocation);
                                
                  }
                  */
             //}

              String outputLocation1 = FolderPath + CellValue_Response + "/PDFExpected/"
                        // Fetch the production response accoding to the Response file and store in a variable 

                        try {
                        ExpectedResponse = new File(""+ FolderPath + CellValue_Response + "/" + testStepName).text

                       

                        xmlHolder1  = (new com.eviware.soapui.support.GroovyUtils( context )).getXmlHolder(ExpectedResponse)
                                                xmlHolder1.namespaces["ns"] = "http://vedaxml.com/vxml2/veda-th-pdf-response-v1.0"
                                                pdfNode1 = xmlHolder1.getNodeValue("//ns:pdf")
                                                               // log.info "pdfNode "+pdfNode
                                                                //XmlHolder xmlHolder = new XmlHolder("AResponse");

                                                             //   log.info "hellooutput" + outputLocation + "\\" + testStepName + ".pdf" 

                         try{
                                 if (pdfNode1 != null) {
                                File dir = new File(outputLocation1);
                                dir.mkdirs();
               // log.info "output" + outputLocation + "\\" + testStepName + ".pdf"  
                                      //PDFFileName =  (testStepName).removeextension 
               
                                     PDFFileName1 = testStepName.substring(0,testStepName.length()-4)  
                                                                    
                                byte[] decodedPdfByteArray = DatatypeConverter.parseBase64Binary(pdfNode);
                                                               
                                                                                log.info "ExpectedpdfNode"
                                                                                BufferedOutputStream bos = new BufferedOutputStream(
                                                                                                                new FileOutputStream(outputLocation1 + "\\" + PDFFileName1 + ".pdf"));
                                                                                bos.write(decodedPdfByteArray, 0, decodedPdfByteArray.length);
                                                                                bos.flush();
                                                                                bos.close();
                                                                                log.info "Expected PDF Generated and Saved Succeccfully"  
                                                         
                                                                }
                         }
                                catch (Exception e) {
                                       e.printStackTrace();
                                }
                  
                        // Compare the variables and print the difference between two response in report
                        def  ResponseDifference=XMLUnit.compareXML(ExpectedResponse, ActualResponse) 
                        def DetailedDiff dd = new DetailedDiff(ResponseDifference)
                         diffCount = 0
                        List DDgetAll = dd.getAllDifferences()
                              for(int k=0; k< DDgetAll.size(); k++){
                                   if (! (DDgetAll.get(k).toString().contains("reportCreateDate") || (DDgetAll.get(k).toString().contains("businessRepresentativesList") || DDgetAll.get(k).toString().contains("thsccor:scoringErrorCode")|| DDgetAll.get(k).toString().contains("fault")  || DDgetAll.get(k).toString().contains("dataProviderId") || DDgetAll.get(k).toString().contains("summaryValue") || DDgetAll.get(k).toString().contains("thtp:description")
                                    || DDgetAll.get(k).toString().contains("thsccor:probabilityAdverse")))){
                                    log.info " "+DDgetAll.get(k)
                                          diffCount = diffCount+1
                                    } 

                              }

                        ResponseDifference=ResponseDifference.toString()            
                        //log.info "Result of the comparision between Responses is :- "+ResponseDifference
  
                        //assert ResponseDifference == "Identical":"Failed" 
                       // log.info "ResponseDifference" +ResponseDifference 
                     // if (!ResponseDifference.contains("identical") && !ResponseDifference.contains("Date")){
                      if ( diffCount > 0)                      
                      {
                                                
                        testRunner.fail("Values do not match")
                              status = "Failed"
                              statusColor = "#ff0000"
                              failCount = failCount+1
                              log.info failCount
                              testCaseStatus.add("\""+testStepName+"\"")
                              testCaseStatus.add("\""+status+"\"")
                        }
                                          
                       else{
                              
                                                                 
                              status = "Passed"
                              statusColor = "#008000"
                              passCount = passCount+1
                              log.info passCount 
                              log.info "Values are Identical"
                         testCaseStatus.add("\""+testStepName+"\"")
                        testCaseStatus.add("\""+status+"\"")
                        }
                  
                  /*
                  catch(Exception e){
                        log.info ("Issue is :- " +e)
                  } */
            

                  // Now delete the run time test step from Test case for reusability 
                  def RemoveTestStep = tc.getTestStepByName(testStepName)
                  log.info "Removing Test Step" +RemoveTestStep
                  if (RemoveTestStep != null) { 
                        tc.removeTestStep(RemoveTestStep)
                  }

                  


                
     //Create an html file for generating report
     //def fileName = new SimpleDateFormat("yyyyMMddhhmm'.txt'").format(new Date());
     def fileName = new SimpleDateFormat("yyyyMMddhhmm'.html'").format(new Date());
     // Defining a file handler/pointer to handle the file. 
    // def inputFile = new File("D:\\Report\\"+fileName)
     
      
      def ReportPathtest = FolderPath + CellValue_Response + "/Report/"

       try{
                                File dir = new File(ReportPathtest);
                                dir.mkdirs();
     // log.info "test" +ReportPathtest
      def inputFile = new File(ReportPathtest + fileName)       
     def newline = System.getProperty("line.separator")  
     inputFile.write('<html><head><meta name="viewport" content="width=device-width, initial-scale=1">'
     +newline+    '<script src="C:/Users/vxs178/Desktop/AutomationTestFrameWork/Report/jquery-1.12.4.min.js"></script>'
     +newline+    '<script src="C:/Users/vxs178/Desktop/AutomationTestFrameWork/Report/jquery.min.js"></script>'
     +newline+    '<script src="C:/Users/vxs178/Desktop/AutomationTestFrameWork/Report/highcharts.js"></script>'
     +newline+    '<link rel="stylesheet" type="text/css" href="C:/Users/vxs178/Desktop/AutomationTestFrameWork/Report/theme.css">'
     +newline+    '<link href="C:/Users/vxs178/Desktop/AutomationTestFrameWork/Report/jquery-ui.css" rel="stylesheet"> '
     +newline+    '<title>Test Status Report</title>'
     +newline+    '</head>'
     +newline+ '<script type="text/javascript">'
     
     +newline+    '$( document ).ready(function() {'           
     +newline+       'var txt="";'
     +newline+       'var sno=1;'
     +newline+       'var testCaseStatus= '+testCaseStatus
     +newline+       'var testCaseSize=' + testCaseStatus.size
    +newline+       'var reportType= '+Report_Type
     +newline+       'var reportTypeSize=' + Report_Type.size              
     +newline+       'var k = 0;'
      
                                               
     +newline+       'for (var m = 0; m < testCaseSize; m++) {'
     +newline+          'var testStepName = testCaseStatus[m]'
     +newline+                'console.log(testStepName)'
     +newline+                'var status = testCaseStatus[m+1]'
     +newline+                'if(status=="Passed"){'
     +newline+                      'sColor="#008000"}'
     +newline+                'else if(status="Failed"){'
     +newline+                      'sColor="#ff0000"}'
     +newline+                'console.log(status)'
     
     +newline+                'txt="<tr><td>"+sno+"</td><td>"+reportType[k]+"</td><td>"+testStepName+"</td><td bgcolor="+sColor+">"+status+"</td></tr>"'      
     +newline+           'if(txt != ""){'
     +newline+                '$("#table").append(txt).removeClass("hidden");'
     +newline+            '}'
     +newline+                ' m=m+1 ;'
     +newline+                ' k=k+1 ;'
     +newline+                ' sno=sno+1 ;'    
     +newline+        '}' 
              
     +newline+    '});'
     
     +newline+    '</script>'
     +newline+    '<script language="JavaScript">'
     +newline+    '$(document).ready(function() {'
     +newline+    'var passCount= '+passCount
     +newline+      'var failCount= '+failCount
     +newline+      'var total= passCount+failCount'
      
     +newline+  '$(function () {'                          
     +newline+    '$("#container").highcharts({'
     +newline+          'chart: {'
     +newline+                      'backgroundColor: "#D6CECE",'
     +newline+                'type: "pie",'
     +newline+                'options3d: {'
     +newline+                      'enabled: true,'
     +newline+                      'alpha: 45'
     +newline+                '}'
     +newline+             '},'
     +newline+             'title: {'
     +newline+                'text: "Test Status Report"'
     +newline+                '},'
     +newline+                'subtitle: {'
     +newline+                      'text: "Test Step Execution"'
     +newline+                '},'
     +newline+                'plotOptions: {'
     +newline+                      'pie: {'
     +newline+                            'innerSize: 100,'
     +newline+                          'depth: 45,'
     +newline+                            'dataLabels: {'
     +newline+                                        'enabled: true,'
     +newline+                                        'format: "<b>{point.name}</b>: {point.percentage:.1f} %",'
     +newline+                                        'style: {'
     +newline+                                  'color: (Highcharts.theme && Highcharts.theme.contrastTextColor) || "Black"'
     +newline+                                  '}'
     +newline+                            '}'
     +newline+                       '}'
     +newline+                   '},'
     +newline+                   'credits: {'
     +newline+                      'enabled: false'
     +newline+                    '},'
     +newline+                    'series: [{'
     +newline+                       'name: "TestStep Result",'
     +newline+                      'data: ['
     +newline+                            '["Pass", passCount],'
     +newline+                          '["Fail", failCount]'
     +newline+                       ']'
     +newline+                     '}]'
     +newline+                   '});'
     +newline+               '});'
     +newline+          '});'
     +newline+ '</script>'
     +newline+ '<body>'
     +newline+ '<div id="container" style="width: 100%; height: 400px;"></div>'
     +newline+ '<table id="table" class="hidden" style="top: 200px" border="1">'
     +newline+ '<tr>'
     +newline+ '<th>Sno.</th>'
     +newline+ '<th>Report Type</th>'
     +newline+ '<th>Test Step Name</th>'  
     +newline+ '<th>Result</th>'
     +newline+ '</tr>'
     +newline+ '</table>'
     +newline+ '</body>'
     +newline+ '</html>')

     log.info "Report generated Successfully at ----------> " +inputFile 
      }
                                catch (Exception e) {
                                      log.info "Exception" +e
                                }
                                 }
                        catch (FileNotFoundException fnfe)
                        {
                                log.info "File Not Found --->" +testStepName
                                 testRunner.fail("Values do not match")
                              status = "Failed"
                              statusColor = "#ff0000"
                              failCount = failCount+1
                              log.info failCount
                              testCaseStatus.add("\""+testStepName+"\"")
                              testCaseStatus.add("\""+status+"\"")
                        }

            }
                        
     }
                  
                 //Create an html file for generating report
     //def fileName = new SimpleDateFormat("yyyyMMddhhmm'.txt'").format(new Date());
     def fileName = new SimpleDateFormat("yyyyMMddhhmm'.html'").format(new Date());
     // Defining a file handler/pointer to handle the file. 
    // def inputFile = new File("D:\\Report\\"+fileName)
     
      
    //  def ReportPathtest = FolderPath + CellValue_Response + "/Report/"
              def ReportPathtest1 = "C:/Users/vxs178/Desktop/AutomationTestFrameWork/Report/"
       try{
                                File dir = new File(ReportPathtest1);
                                dir.mkdirs();
     // log.info "test" +ReportPathtest
      def inputFile = new File(ReportPathtest1 + fileName)       
     def newline = System.getProperty("line.separator")  
     inputFile.write('<html><head><meta name="viewport" content="width=device-width, initial-scale=1">'
     +newline+    '<script src="C:/Users/vxs178/Desktop/AutomationTestFrameWork/Report/jquery-1.12.4.min.js"></script>'
     +newline+    '<script src="C:/Users/vxs178/Desktop/AutomationTestFrameWork/Report/jquery.min.js"></script>'
     +newline+    '<script src="C:/Users/vxs178/Desktop/AutomationTestFrameWork/Report/highcharts.js"></script>'
     +newline+    '<link rel="stylesheet" type="text/css" href="C:/Users/vxs178/Desktop/AutomationTestFrameWork/Report/theme.css">'
     +newline+    '<link href="C:/Users/vxs178/Desktop/AutomationTestFrameWork/Report/jquery-ui.css" rel="stylesheet"> '
     +newline+    '<title>Test Status Report</title>'
     +newline+    '</head>'
     +newline+ '<script type="text/javascript">'
     
     +newline+    '$( document ).ready(function() {'           
     +newline+       'var txt="";'
     +newline+       'var sno=1;'
     +newline+       'var testCaseStatus= '+testCaseStatus
     +newline+       'var testCaseSize=' + testCaseStatus.size
    +newline+       'var reportType= '+Report_Type
     +newline+       'var reportTypeSize=' + Report_Type.size              
     +newline+       'var k = 0;'
      
                                               
     +newline+       'for (var m = 0; m < testCaseSize; m++) {'
     +newline+          'var testStepName = testCaseStatus[m]'
     +newline+                'console.log(testStepName)'
     +newline+                'var status = testCaseStatus[m+1]'
     +newline+                'if(status=="Passed"){'
     +newline+                      'sColor="#008000"}'
     +newline+                'else if(status="Failed"){'
     +newline+                      'sColor="#ff0000"}'
     +newline+                'console.log(status)'
     
     +newline+                'txt="<tr><td>"+sno+"</td><td>"+reportType[k]+"</td><td>"+testStepName+"</td><td bgcolor="+sColor+">"+status+"</td></tr>"'      
     +newline+           'if(txt != ""){'
     +newline+                '$("#table").append(txt).removeClass("hidden");'
     +newline+            '}'
     +newline+                ' m=m+1 ;'
     +newline+                ' k=k+1 ;'
     +newline+                ' sno=sno+1 ;'    
     +newline+        '}' 
              
     +newline+    '});'
     
     +newline+    '</script>'
     +newline+    '<script language="JavaScript">'
     +newline+    '$(document).ready(function() {'
     +newline+    'var passCount= '+passCount
     +newline+      'var failCount= '+failCount
     +newline+      'var total= passCount+failCount'
      
     +newline+  '$(function () {'                          
     +newline+    '$("#container").highcharts({'
     +newline+          'chart: {'
     +newline+                      'backgroundColor: "#D6CECE",'
     +newline+                'type: "pie",'
     +newline+                'options3d: {'
     +newline+                      'enabled: true,'
     +newline+                      'alpha: 45'
     +newline+                '}'
     +newline+             '},'
     +newline+             'title: {'
     +newline+                'text: "Test Status Report"'
     +newline+                '},'
     +newline+                'subtitle: {'
     +newline+                      'text: "Test Step Execution"'
     +newline+                '},'
     +newline+                'plotOptions: {'
     +newline+                      'pie: {'
     +newline+                            'innerSize: 100,'
     +newline+                          'depth: 45,'
     +newline+                            'dataLabels: {'
     +newline+                                        'enabled: true,'
     +newline+                                        'format: "<b>{point.name}</b>: {point.percentage:.1f} %",'
     +newline+                                        'style: {'
     +newline+                                  'color: (Highcharts.theme && Highcharts.theme.contrastTextColor) || "Black"'
     +newline+                                  '}'
     +newline+                            '}'
     +newline+                       '}'
     +newline+                   '},'
     +newline+                   'credits: {'
     +newline+                      'enabled: false'
     +newline+                    '},'
     +newline+                    'series: [{'
     +newline+                       'name: "TestStep Result",'
     +newline+                      'data: ['
     +newline+                            '["Pass", passCount],'
     +newline+                          '["Fail", failCount]'
     +newline+                       ']'
     +newline+                     '}]'
     +newline+                   '});'
     +newline+               '});'
     +newline+          '});'
     +newline+ '</script>'
     +newline+ '<body>'
     +newline+ '<div id="container" style="width: 100%; height: 400px;"></div>'
     +newline+ '<table id="table" class="hidden" style="top: 200px" border="1">'
     +newline+ '<tr>'
     +newline+ '<th>Sno.</th>'
     +newline+ '<th>Report Type</th>'
     +newline+ '<th>Test Step Name</th>'  
     +newline+ '<th>Result</th>'
     +newline+ '</tr>'
     +newline+ '</table>'
     +newline+ '</body>'
     +newline+ '</html>')

     log.info "Report generated Successfully at ----------> " +inputFile 
      }
                                catch (Exception e) {
                                log.info "Exception"  +e
                                }
                                 
                       
     
