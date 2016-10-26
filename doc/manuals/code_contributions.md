# Sample code

This section contains code snippets (most of them contributed by
external developers) that can be used as examples for programming with
Orion Context Broker in different technologies. Note that Orion Context
Broker evolves with time so it could happen that the code examples get
obsolete in some moment (the publication date is provides as reference).

## Java

Thanks to Kurento (http://www.kurento.org) a Java library that shows how
to access to Orion Context Broker:
<https://github.com/KurentoLegacy/kmf-orion-connector/tree/develop>
(published June 2014).

Thanks to Alejandro Villamarin (published around October 2013):

       import com.sun.jersey.api.client.Client;
       import com.sun.jersey.api.client.ClientResponse;
       import com.sun.jersey.api.client.WebResource;

       //Test the Orion Broker, query for Room
       String urlString = "http://orion.lab.fiware.org:1026/v1/queryContext";
       String XML = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>" + 
            "<queryContextRequest>" + 
            "<entityIdList>" + 
            "<entityId type=\"Room\" isPattern=\"true\">" + 
            "<id>Room.*</id>" + 
            "</entityId>" +
            "</entityIdList>" + 
            "<attributeList>" + 
            "<attribute>temperature</attribute>" + 
            "</attributeList>" + 
            "</queryContextRequest>";      
       try {     
          Client client = Client.create();
          WebResource webResource = client.resource(urlString);
          ClientResponse response = webResource.type("application/xml").header("X-Auth-Token", "your_auth_token").post(ClientResponse.class, XML);
       
          if (response.getStatus() != 200) {
             System.err.println(response.getEntity(String.class));
        throw new RuntimeException("Failed : HTTP error code : " + response.getStatus());
          }
       
          System.out.println("Output from Server .... \n");
          String output = response.getEntity(String.class);
          System.out.println(output);
       
       } 
       catch (Exception e) {
          System.err.println("Failed. Reason is " + e.getMessage());
       }

## JavaScript

Using JQuery AJAX, thanks to Marco Vereda Manchego ([original
post](http://marcovereda.blogspot.com.es/2013/10/code-snippet-for-custom-http-post.html))
(published around October 2013):

     function capture_sensor_data(){  
     var contentTypeRequest = $.ajax({  
          url: 'http://orion.lab.fiware.org:1026/v1/queryContext',  
          data: '<?xml version="1.0" encoding="UTF-8"?>  
                    <queryContextRequest>  
                         <entityIdList>  
                              <entityId type = "Sensor" isPattern="true">  
                                   <id>urn:smartsantander:testbed:.*</id>   
                              </entityId>   
                         </entityIdList>   
                         <attributeList>   
                              <attribute>Latitud</attribute>  
                              <attribute>Longitud</attribute>  
                         </attributeList>       
                    </queryContextRequest>  ',  
          type: 'POST',  
          dataType: 'xml',  
          contentType: 'application/xml',  
          headers: { 'X-Auth-Token' :   
               'you_auth_token'}  
          });  
          contentTypeRequest.done(function(xml){   
               var latitud = new Array();  
               var longitud = new Array();  
               var i = 0;       
               var len = $(xml).find("contextAttribute").children().size();  
               $(xml).find('contextAttribute').each(function(){  
                         if (  $(this).find('type').text() == "urn:x-ogc:def:phenomenon:IDAS:1.0:latitude"  
                               && $(this).find('contextValue').text() != "" )  
                         {   
                              latitud[i] = $(this).find('contextValue').text(); i=i+1;  
                         }  
                         if ($(this).find('type').text() == "urn:x-ogc:def:phenomenon:IDAS:1.0:longitude"  
                                && $(this).find('contextValue').text() != ""       )  
                         {   
                              longitud[i] = $(this).find('contextValue').text();   
                         }  
                                                                      });  
               for (var j=0; j<i; j++){                 
                    console.log("DEBUG :" + latitud[j] + "," + longitud[j]);                 
               }                      
          });                 
          contentTypeRequest.fail(function(jqXHR, textStatus){     
               console.log( "DEBUG :  Ajax request failed... (" + textStatus + ' - ' + jqXHR.responseText + ")." );  
          });  
          contentTypeRequest.always(function(jqXHR, textStatus){       });  
     }

## Arduino

Thanks to Enrique Hernandez Zurita, using Orion 0.11.0 and 0.12.0
(published around June 2014):

    #include <SPI.h>
    #include <WiFi.h>
    #include <WString.h>
    char ssid[] = "SSID"; 
    char pass[] = "CONTRASEÑA";    
              
    int status = WL_IDLE_STATUS;
    IPAddress server(130,206,82,115);
    WiFiClient client;
    String line="";
    int value=0;
    int led=9;

    void setup() {
      
        pinMode(led,OUTPUT);
        Serial.begin(9600); 
        while (!Serial) {  ;
        }
        if (WiFi.status() == WL_NO_SHIELD) {

        Serial.println("WiFi shield not present"); 
        while(true);
        } 
        while (status != WL_CONNECTED) { 
        Serial.print("Attempting to connect to SSID: ");
        Serial.println(ssid); 
        status = WiFi.begin(ssid, pass);
        delay(5000); 
        } 
        Serial.println("Connected to wifi");
        Serial.println("\nStarting connection to server..."); 
        
        if (client.connect(server, 1026)) {
        Serial.println("connected to server");
        client.println("POST /NGSI10/queryContext HTTP/1.1");
        client.println("Host: 130.206.82.115:1026");
        client.println("User-Agent: Arduino/1.1");
        client.println("Connection: close");
        client.println("Content-Type: application/xml");
        client.print("Content-Length: ");
        client.println("227");
        client.println();
        client.println("<?xml version=\"1.0\" encoding=\"UFT-8\"?>");
        client.println("<queryContextRequest>");
        client.println("<entityIdList>");
        client.println("<entityId type=\"UPCT:ACTUATOR\" isPattern=\"false\">");
        client.println("<id>UPCT:ACTUATOR:1</id>");
        client.println("</entityId>");
        client.println("</entityIdList>");
        client.println("<attributeList/>");
        client.println("</queryContextRequest>");
        client.println();
        
        Serial.println("XML Sent");
        
        }else{Serial.println("Impossible to connect");}
    }

    void loop() {
        if(value=1){digitalWrite(led, HIGH);}
      
        while (client.available()) {
        char c = client.read();
        Serial.write(c);
        line=line + c;
            if(c==10){
            value=searchValue(line,value);
            line="";
            }
         }
      
         if (!client.connected()) {
         Serial.println();
         Serial.println("disconnecting from server.");
         Serial.println(value);
         client.stop();
         while(true);
         }
    }


    int searchValue(String s,int i) {
      int beginning,ending;
      String val;
      beginning=s.indexOf('>');
      ending=s.indexOf('<', beginning+1);
        if(s.startsWith("contextValue",s.indexOf('<')+1)){
        val =s.substring(beginning+1,ending);
        return val.toInt();
        }else{return i;}
    }
    
    
    
## Python

       import requests
       import json

       CB_HOST = "127.0.0.1"  #change to this ip "192.168.56.101" if you are using the context broker configured at VM
       CB_PORT = "1026"
       CB_URL = "http://" + CB_HOST+ ":" + CB_PORT
       POST_URL = CB_URL + "/v2/entities"
       HEADERS = {'content-type': 'application/json','accept': 'application/json'} #NOT NOW 'Fiware-Service': CB_FIWARE_SERVICE ,'Fiware-ServicePath': CB_FIWARE_SERVICEPATH,'X-Auth-Token' : TOKEN}


       # fiware version - just for test
       def fiware_version():
           r = requests.get(CB_URL/version)
           print r.text

       # test
       def test_version():
           fiware_version()


       # register entities
       def register_entities():
           print "WELCOME TO FIWARE WORLD! LET'S TEST THE ORION CONTEXT BROKER GENERIC ENABLER!\n"
           print "Let's register some entities...\n"
           n_entities = input("Enter the number of entities you want to register: ")

           for e in range(0, n_entities):
              print "######################################################"
              ENTITY_ID = raw_input("Enter the ID of the entity %d: " % (e+1))
              ENTITY_TYPE = raw_input("Enter the type of the entity %d: " % (e+1))
              n_attrs = input("Enter the number of attributes this entity has: ")

              PAYLOAD = '{ \
                     "id": "'+ENTITY_ID+'", \
                     "type": "'+ENTITY_TYPE+'", '\

              print "------------------------------------------------------------"
       for a in range(0, n_attrs):
           ATTR_NAME = raw_input("Enter the name of the attribute %d: " % (a+1))
           ATTR_TYPE = raw_input("Enter the type of the attribute %d: " % (a+1))
           ATTR_VALUE = raw_input("Enter the value of the attribute %d: " % (a+1))
           #TODO: add metadata handling...

           PAYLOAD += '"'+ATTR_NAME+'": {  \
                          "type": "'+ATTR_TYPE+'", \
                          "value": "'+ATTR_VALUE+'"' \
                      + ('}, ' if (a != n_attrs-1) else '}  }')


       print "------------------------------------------------------------"
       
        print json.dumps(json.loads(PAYLOAD), indent=4)
       print "This entity is being registered..."
       r = requests.post(POST_URL, data=PAYLOAD, headers=HEADERS)

       print "The status of this operation is: " + str(r.status_code) + (" OK! Successful register!"
            if r.status_code==201 or r.status_code==200 else " Some error occurred!")
       print r.text



       if __name__ == "__main__":
           #test_version()
           register_entities()


