## Zebra-Printer_Sample program of Send And Recieve Data from Winows (PC, Java)
## サンプルプログラム：Windows/Javaでプリンタとデータ送受信する


> テスト環境
- windows 11
- java 20
- Zebra ZQ630R, ZD611R
- Link-OS SDK 2024 Apr

> 主な機能
- ZPLの送信
- ZPL送信に対してのフィードバック受信（^HV)
- SGDの送受信
<br>

> サンプルコード

```java
import com.zebra.sdk.comm.ConnectionException;
import com.zebra.sdk.comm.DriverPrinterConnection;
import com.zebra.sdk.comm.TcpConnection;
import com.zebra.sdk.comm.Connection;
import com.zebra.sdk.printer.SGD;
import com.zebra.sdk.printer.discovery.DiscoveredPrinter;
import com.zebra.sdk.printer.discovery.DiscoveredPrinterDriver;
import com.zebra.sdk.printer.discovery.DiscoveryException;
import com.zebra.sdk.printer.discovery.DiscoveryHandler;
import com.zebra.sdk.printer.discovery.NetworkDiscoverer;
import com.zebra.sdk.printer.discovery.UsbDiscoverer;

import java.time.Duration;
import java.time.LocalDateTime;
import java.time.Period;
import java.util.ArrayList;
import java.util.List;


public class App {

    String printerIpAdr = "192.168.4.58";
    static String printerDriverName1 = "ZDesigner ZQ620 (ZPL)";
    static String printerDriverName2 = "ZDesigner ZD611R-300dpi ZPL";

    // ZPL: Send ZPL + Recieve RFID Feedback Data
    static String zplData1 = """
        ^XA
        ^DFR:AAA.ZPL
        ^FO20,120^A0N,60^FN1^FS
        ^HV1,,output:^FS
        ^XZ
        ^XA
        ^XFR:AAA.ZPL
        ^FN1
        ^FD
        AAAAA
        ^FS
        ^XZ
        """;


    // ZPL: Send ZPL + Recieve RFID Feedback Data
    static String zplData2 = """
        ^XA
        ^RS8
        ^RFR,H,0,8,2^FN1^FS
        ^FH_^HV1,,8-byte Tag ID Data:[,]_0D_0A^FS
        ^RFR,H^FN2^FS
        ^FH_^HV2,,All EPC Data:[,]_0D_0A^FS
        ^XZ        
        """;

    // ZPL: Send ZPL
    static String zplData3 = """
    ^XA
    ^RS8
    ^RFR,H,0,8,2^FS
    ^RFR,H^FS
    ^XZ        
    """;



    public static void main(String[] args) throws Exception {

        LocalDateTime startTime;
        LocalDateTime endTime;
        Duration duration;

        
        System.out.println("Hello, Zebra!");

        // ディスカバリー処理（ネットワークプリンタ）
        // new App().discoverNetworkPrinters();

        // ディスカバリー処理（ローカルプリンタ）
        // new App().discoverLocalPrinters();


        // ------------------------------
        // Sample Code 1
        // Open > SGD  > Close > Open > ZPL > Close
        // ------------------------------

        System.out.println("*********************************");
        System.out.println("Sample Code 1");
        System.out.println("*********************************");
        
        startTime = LocalDateTime.now();
        System.out.print(startTime + " : ");
        System.out.println("*** Program Start ***");

        // Clear rfid log entries
        new App().setvarOverUsb(printerDriverName2, "rfid.log.clear", "");
        //String getStr2 = new App().getvarOverUsb(printerDriverName2, "rfid.log.entries");
        //System.out.print(LocalDateTime.now() + " : ");
        //System.out.println("\"rfid.log.entries\" are ");
        //System.out.println(getStr2);
        
        // Configure rfid logs
        //new App().setvarOverUsb(printerDriverName2, "rfid.log.enabled", "no");
        //String getStr1 = new App().getvarOverUsb(printerDriverName2, "rfid.log.enabled");
        //System.out.print(LocalDateTime.now() + " : ");
        //System.out.print("\"rfid.log.enabled\" is ");
        //System.out.println(getStr1);

        // データ送受信（USB）
        System.out.print(LocalDateTime.now() + " : ");
        System.out.println("Start ZPL.");
        new App().sendRecieveZplOverUsb(printerDriverName2, zplData2);
        System.out.print(LocalDateTime.now() + " : ");
        System.out.println("End ZPL.");

        // Get log.entries
        String getStr3 = new App().getvarOverUsb(printerDriverName2, "rfid.log.entries");
        System.out.print(LocalDateTime.now() + " : ");
        System.out.println("\"rfid.log.entries\" are ");
        System.out.println(getStr3);


        // Clear rfid log entries
        //new App().setvarOverUsb(printerDriverName2, "rfid.log.clear", "");
        //System.out.print(LocalDateTime.now() + " : ");
        //System.out.println("rfid logs are cleared.");

        endTime = LocalDateTime.now();
        System.out.print(endTime + " : ");
        System.out.println("*** Program End ***");

        duration = Duration.between(startTime, endTime);
        System.out.println("Duration time is " + duration);

        // ------------------------------
        // Sample Code 2
        // Open > SGD + ZPL > Close
        // ------------------------------
        System.out.println("*********************************");
        System.out.println("Sample Code 2");
        System.out.println("*********************************");

        startTime = LocalDateTime.now();
        System.out.print(startTime + " : ");
        System.out.println("*** Program Start ***");

        new App().sendRecieveZplOverUsb_v2(printerDriverName2, zplData3);

        endTime = LocalDateTime.now();
        System.out.print(endTime + " : ");
        System.out.println("*** Program End ***");

        duration = Duration.between(startTime, endTime);
        System.out.println("Duration time is " + duration);


        
        // ------------------------------
        // Sample Code 3
        // Open > ZPL > Close
        // ------------------------------
        System.out.println("*********************************");
        System.out.println("Sample Code 3");
        System.out.println("*********************************");
        startTime = LocalDateTime.now();
        System.out.print(startTime + " : ");
        System.out.println("*** Program Start ***");

        new App().sendRecieveZplOverUsb(printerDriverName2, zplData2);

        endTime = LocalDateTime.now();
        System.out.print(endTime + " : ");
        System.out.println("*** Program End ***");

        duration = Duration.between(startTime, endTime);
        System.out.println("Duration time is " + duration);
        

    }

    // ディスカバリー処理（ローカルプリンタ）
    public void discoverLocalPrinters(){
        try {
            for (DiscoveredPrinterDriver dPrinter : UsbDiscoverer.getZebraDriverPrinters()) {
                System.out.println(dPrinter);    
            }
        } catch (ConnectionException e) {
            System.out.println("Error discovering local printers: " + e.getMessage());
        }

        System.out.println("Done discovering local printers.");
    }

    // ディスカバリー処理（ネットワークプリンタ）
    public void discoverNetworkPrinters(){
    
        List<DiscoveredPrinter> printers = new ArrayList<DiscoveredPrinter>();

        // ハンドラ処理 
        DiscoveryHandler discoveryHandler = new DiscoveryHandler() {

            // プリンタ検知時
            public void foundPrinter(DiscoveredPrinter printer) {
                printers.add(printer);
            }
            
            // Discovery 処理修了時
            public void discoveryFinished() {
                for (DiscoveredPrinter printer : printers) {
                    System.out.println(printer);
                }
                System.out.println("Discovered " + printers.size() + " printers.");
            }

            // Error発生時
            public void discoveryError(String message) {
                System.out.println("An error occurred during discovery : " + message);
            }
        };

        // ハンドラ処理の実行（Discovery処理）
        try {
            System.out.println("Starting printer discovery.");
            NetworkDiscoverer.findPrinters(discoveryHandler);
            
        } catch (DiscoveryException e) {
            e.printStackTrace();
        }

        /* 
        // ハンドラ処理の実行（特定のIPアドレスをDiscovery）
        try {
            NetworkDiscoverer.directedBroadcast(discoveryHandler,printerIpAdr);
        } catch (DiscoveryException e) {
            e.printStackTrace();
        }
        */


    }


        // SETVAR（USB）
        public void setvarOverUsb(String printerName, String sgdConfig, String sgdArgs) throws ConnectionException {
        
            // Instantiate connection for ZPL USB port for given printer name
            Connection thePrinterConn = new DriverPrinterConnection(printerName);
    
            try {
                // Open the connection - physical connection is established here.
                thePrinterConn.open();
    
                // Configuration 
                SGD.SET(sgdConfig, sgdArgs, thePrinterConn);

            } catch (ConnectionException e) {
                // Handle communications error here.
                e.printStackTrace();
            } finally {
                // Close the connection to release resources.
                thePrinterConn.close();
            }
        }    

        // GETVAR（USB）
        public String getvarOverUsb(String printerName, String sgdConfig) throws ConnectionException {
            
            String getStr = null;

            // Instantiate connection for ZPL USB port for given printer name
            Connection thePrinterConn = new DriverPrinterConnection(printerName);
    
            try {
                // Open the connection - physical connection is established here.
                thePrinterConn.open();
    
                // Configuration 
                getStr = SGD.GET(sgdConfig, thePrinterConn);
                // System.out.println(getStr);

            } catch (ConnectionException e) {
                // Handle communications error here.
                e.printStackTrace();
            } finally {
                // Close the connection to release resources.
                thePrinterConn.close();
            }

            return getStr;
        }    

        /* 
        // SGD送信（USB）
        public void sgdOverUsb(String printerName) throws ConnectionException {
        
            // Instantiate connection for ZPL USB port for given printer name
            Connection thePrinterConn = new DriverPrinterConnection(printerName);
    
            try {
                // Open the connection - physical connection is established here.
                thePrinterConn.open();
    
                // Configuration 
                SGD.SET("rfid.log.enabled", "yes", thePrinterConn);
                SGD.SET("rfid.log.clear", "", thePrinterConn);
                
                // Check SGD
                String logEnabled = SGD.GET("rfid.log.enabled", thePrinterConn);
                //String logClear = SGD.GET("rfid.log.clear", thePrinterConn);
                String logEntries = SGD.GET("rfid.log.entries", thePrinterConn);

                System.out.println("rfid.log.enabled is [" + logEnabled + "]");
                System.out.println("rfid logs are cleared.");
                System.out.println("rfid.log.entries is [" + logEntries + "]");

            } catch (ConnectionException e) {
                // Handle communications error here.
                e.printStackTrace();
            } finally {
                // Close the connection to release resources.
                thePrinterConn.close();
            }
        }    
        */
    

    // データ送信（USB）
    public void sendZplOverUsb(String printerName, String zplData) throws ConnectionException {
        
        // Instantiate connection for ZPL USB port for given printer name
        Connection thePrinterConn = new DriverPrinterConnection(printerName);

        try {
            // Open the connection - physical connection is established here.
            thePrinterConn.open();

            // ByteArrayでZPLを送信
            thePrinterConn.write(zplData.getBytes());

        } catch (ConnectionException e) {
            // Handle communications error here.
            e.printStackTrace();
        } finally {
            // Close the connection to release resources.
            thePrinterConn.close();
        }
    }    
    
    // データ受信（USB）
    public String RecieveDataOverUsb(String printerName) throws ConnectionException {
        
        // Instantiate connection for ZPL USB port for given printer name
        Connection thePrinterConn = new DriverPrinterConnection(printerName);

        try {
            // Open the connection - physical connection is established here.
            thePrinterConn.open();

            // ByteArrayで受信データを受信
            byte[] b = thePrinterConn.read();
            String recieveData = new String(b);
            //System.out.println(recieveString);
            return recieveData;

        } catch (ConnectionException e) {
            // Handle communications error here.
            e.printStackTrace();
        } finally {
            // Close the connection to release resources.
            thePrinterConn.close();
        }

        return null;
    }

    // データ送受信（USB）
    public String sendRecieveZplOverUsb(String printerName, String zplData) throws ConnectionException, InterruptedException {
        
        String recieveString = null;

        // Instantiate connection for ZPL USB port for given printer name
        Connection thePrinterConn = new DriverPrinterConnection(printerName);

        try {
            // Open the connection - physical connection is established here.
            thePrinterConn.open();


            // ByteArrayでZPLを送信
            thePrinterConn.write(zplData.getBytes());

            // 必要に応じて入れる
            // Thread.sleep(1000);

            // ByteArrayで受信データを受信
            byte[] b = thePrinterConn.read();
            recieveString = new String(b);
            System.out.println("Recieved data : " + recieveString);
            return recieveString;

        } catch (ConnectionException e) {
            // Handle communications error here.
            e.printStackTrace();
        } finally {
            // Close the connection to release resources.
            thePrinterConn.close();
        }

        return recieveString;
    }



    // データ送受信（USB）
    public void sendRecieveZplOverUsb_v2(String printerName, String zplData) throws ConnectionException, InterruptedException {
        
        
        // Instantiate connection for ZPL USB port for given printer name
        Connection thePrinterConn = new DriverPrinterConnection(printerName);

        try {
            // Open the connection - physical connection is established here.
            thePrinterConn.open();

            SGD.SET("rfid.log.clear", "", thePrinterConn);
            //System.out.print(LocalDateTime.now() + " : ");
            //System.out.println( "rfid.log.entries are ");
            //System.out.println( SGD.GET("rfid.log.entries", thePrinterConn));

            //SGD.SET("rfid.log.enabled", "yes", thePrinterConn);
            //System.out.print(LocalDateTime.now() + " : ");
            //System.out.println( "rfid.log.enabled are " + SGD.GET("rfid.log.enabled", thePrinterConn));

            System.out.print(LocalDateTime.now() + " : ");
            System.out.println("Start ZPL.");

            // ByteArrayでZPLを送信
            thePrinterConn.write(zplData.getBytes());


            // ByteArrayで受信データを受信
            //byte[] b = thePrinterConn.read();
            //String recieveString = new String(b);
            //System.out.println(recieveString);

            System.out.print(LocalDateTime.now() + " : ");
            System.out.println("End ZPL.");

            // 必要に応じて入れる
            Thread.sleep(1000);

            System.out.print(LocalDateTime.now() + " : ");
            System.out.println( "rfid.log.entries are ");
            System.out.println( SGD.GET("rfid.log.entries", thePrinterConn));
            SGD.SET("rfid.log.clear", "", thePrinterConn);


        } catch (ConnectionException e) {
            // Handle communications error here.
            e.printStackTrace();
        } finally {
            // Close the connection to release resources.
            thePrinterConn.close();
        }

    }


    // データ送信（Network）
    public void sendZplOverTcp(String theIpAddress) throws ConnectionException {
         // Instantiate connection for ZPL TCP port at given address
        Connection thePrinterConn = new TcpConnection(theIpAddress, TcpConnection.DEFAULT_ZPL_TCP_PORT);
        try {
            // Open the connection - physical connection is established here.
            thePrinterConn.open();

            // This example prints "This is a ZPL test." near the top of the label.
            String zplData = "^XA^FO20,20^A0N,25,25^FDThis is a ZPL test.^FS^XZ";

            // Send the data to printer as a byte array.
            thePrinterConn.write(zplData.getBytes());
        } catch (ConnectionException e) {
            // Handle communications error here.
            e.printStackTrace();
        } finally {
            // Close the connection to release resources.
            thePrinterConn.close();
        }
    }


}
```
<br>
<br>

```java
Hello, Zebra!
*********************************
Sample Code 1
*********************************
2024-05-19T17:42:38.827760700 : *** Program Start ***
2024-05-19T17:42:39.024936900 : Start ZPL.
Recieved data : 8-byte Tag ID Data:[E20034120131F300]
All EPC Data:[AAAA12345678901234567890]

2024-05-19T17:42:40.453086800 : End ZPL.
2024-05-19T17:42:48.734354300 : "rfid.log.entries" are 
<start>
May-19-2024 17:42:40,R,B7,A1,18,00000000,E20034120131F300
May-19-2024 17:42:40,R,B7,A1,18,00000000,AAAA12345678901234567890       
<end>


2024-05-19T17:42:48.736363700 : *** Program End ***
Duration time is PT9.908603S ★
*********************************
Sample Code 2
*********************************
2024-05-19T17:42:48.737965100 : *** Program Start ***
2024-05-19T17:42:48.747976600 : Start ZPL.
2024-05-19T17:42:48.754979100 : End ZPL.
2024-05-19T17:42:49.760796 : rfid.log.entries are
<start>
May-19-2024 17:42:50,R,B7,A1,18,00000000,E20034120133F300
May-19-2024 17:42:50,R,B7,A1,18,00000000,AAAA12345678901234567890
<end>


2024-05-19T17:42:58.031796500 : *** Program End ***
Duration time is PT9.2938314S ★

*********************************
Sample Code 3
*********************************
2024-05-19T17:42:58.032796 : *** Program Start ***
Recieved data : 8-byte Tag ID Data:[E20034120134F300]
All EPC Data:[AAAA12345678901234567890]

2024-05-19T17:42:59.466641500 : *** Program End ***
Duration time is PT1.4338455S ★

```
