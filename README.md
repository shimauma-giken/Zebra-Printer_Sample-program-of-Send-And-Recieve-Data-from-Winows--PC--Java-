# Zebra-Printer_Sample program of Send And Recieve Data from Winows (PC, Java)
 サンプルプログラム：Windows/Javaでプリンタとデータ送受信する


```java
import com.zebra.sdk.comm.ConnectionException;
import com.zebra.sdk.comm.DriverPrinterConnection;
import com.zebra.sdk.comm.TcpConnection;
import com.zebra.sdk.comm.Connection;

import com.zebra.sdk.printer.discovery.DiscoveredPrinter;
import com.zebra.sdk.printer.discovery.DiscoveredPrinterDriver;
import com.zebra.sdk.printer.discovery.DiscoveryException;
import com.zebra.sdk.printer.discovery.DiscoveryHandler;
import com.zebra.sdk.printer.discovery.NetworkDiscoverer;
import com.zebra.sdk.printer.discovery.UsbDiscoverer;


import java.util.ArrayList;
import java.util.List;


public class App {

    String printerIpAdr = "192.168.4.58";
    static String printerDriverName1 = "ZDesigner ZQ620 (ZPL)";
    static String printerDriverName2 = "ZDesigner ZD611R-300dpi ZPL";
    // ZPL: Recieve Variable Feedback Data
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


    // ZPL: Recieve RFID Feedback Data
    static String zplData2 = """
        ^XA
        ^RS8
        ^RFR,H,0,8,2^FN1^FS
        ^FH_^HV1,,8-byte Tag ID Data:[,]_0D_0A^FS
        ^RFR,H^FN2^FS
        ^FH_^HV2,,All EPC Data:[,]_0D_0A^FS
        ^XZ        
        """;
    
    


    public static void main(String[] args) throws Exception {
        
        System.out.println("Hello, Zebra!");

        // ディスカバリー処理（ネットワークプリンタ）
        new App().discoverNetworkPrinters();

        // ディスカバリー処理（ローカルプリンタ）
        new App().discoverLocalPrinters();

        // データ送受信（USB）
        new App().sendRecieveZplOverUsb(printerDriverName2, zplData2);
        
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
        
        // Instantiate connection for ZPL USB port for given printer name
        Connection thePrinterConn = new DriverPrinterConnection(printerName);

        try {
            // Open the connection - physical connection is established here.
            thePrinterConn.open();


            // ByteArrayでZPLを送信
            thePrinterConn.write(zplData.getBytes());

            Thread.sleep(1000);

            // ByteArrayで受信データを受信
            byte[] b = thePrinterConn.read();
            String recieveString = new String(b);
            System.out.println(recieveString);
            return recieveString;

        } catch (ConnectionException e) {
            // Handle communications error here.
            e.printStackTrace();
        } finally {
            // Close the connection to release resources.
            thePrinterConn.close();
        }

        return "";
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
