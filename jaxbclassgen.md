## JAXB for simple Java-XML serialization ##

There're a number of way to do XML serialization in Java. If you want fine-grained control over 
parsing and serialization you can go for SAX, DOM, or Stax for better performance. 
Yet, what I often want to do is a simple mapping between POJOs and XML. However, 
creating Java classes to do XML event parsing manually is not trivial. 
I recently found JAXB to be a quick and convenient Java-XML mapping or serialization. 

JAXB contains a lot of useful features, you can check out the reference implementation here. 
Kohsuke's blog is also a good resource to learn more about JAXB. For this blog entry, 
I'll show you how to do a simple Java-XML serialization with JAXB. 

### POJO to XML ###

Let's say I have an Item Java object. I want to serialize an Item object to XML format. What I have to do first is to annotate this POJO with a few XML annotation from javax.xml.bind.annotation.* package. See code listing 1 for Item.java 

```java
From the code 
@XmlRootElement(name="Item") indicates that I want <Item> to be the root element. 
@XmlType(propOrder = {"name", "price"}) indicates the order that I want the element to be arranged in XML output. 
@XmlAttribute(name="id", ...) indicates that id is an attribute to <Item> root element. 
@XmlElement(....) indicates that I want price and name to be element within Item.
```

My Item.java is ready. I can then go ahead and create JAXB script for marshaling Item. 

```java
//creating Item data object 
Item item = new Item(); 
item.setId(2); 
item.setName("Foo"); 
item.setPrice(200); 
.....
```

```java
JAXBContext context = JAXBContext.newInstance(item.getClass()); 
Marshaller marshaller = context.createMarshaller(); 
marshaller.marshal(item, new FileWriter("item.xml")); //I want to save the output file to item.xml
```

For complete code Listing please see Code Listing 2 (main.java). The output Code Listing 3 item.xml file 
is created. It looks like this: 

```java
<?xml version="1.0" encoding="UTF-8" standalone="yes"?> 
<ns1:item ns1:id="2" xmlns:ns1="http://blogs.sun.com/teera/ns/item"> 

<ns1:itemName>Foo</ns1:itemName> 
<ns1:price>200</ns1:price> 

</ns1:item>
```

Easy right? You can alternatively channel the output XML as text String, Stream, Writer, 
ContentHandler, etc by simply change the parameter of the marshal(...) method like 

... 
JAXBContext context = JAXBContext.newInstance(item.getClass()); 
Marshaller marshaller = context.createMarshaller(); 
marshaller.marshal(item, <java.io.OutputStream instance>); // save xml output to the OutputStream instance 

... 
JAXBContext context = JAXBContext.newInstance(item.getClass()); 
Marshaller marshaller = context.createMarshaller(); 
StringWriter sw = new StringWriter(); 
marshaller.marshal(item, sw); //save to StringWriter, you can then call sw.toString() to get java.lang.String 

XML to POJO 

Let's reverse the process. Assume that I now have a piece of XML string data and I want to turn it into Item.java object. XML data (Code listing 3) looks like 

<?xml version="1.0" encoding="UTF-8" standalone="yes"?> 
<ns1:item ns1:id="2" xmlns:ns1="http://blogs.sun.com/teera/ns/item"> 
<ns1:itemName>Bar</ns1:itemName> 
<ns1:price>80</ns1:price> 
</ns1:item> 

I can then unmarshal this xml code to Item object by 

... 
ByteArrayInputStream xmlContentBytes = new ByteArrayInputStream (xmlContent.getBytes()); 
JAXBContext context = JAXBContext.newInstance(Item.getClass()); 
Unmarshaller unmarshaller = context.createUnmarshaller(); 
//note: setting schema to null will turn validator off 
unmarshaller.setSchema(null); 
Object xmlObject = Item.getClass().cast(unmarshaller.unmarshal(xmlContentBytes)); 
return xmlObject; 
... 

For complete code Listing please see Code Listing 2 (main.java). The XML source can come in many forms both from Stream and file. The only difference, again, is the method parameter: 

... 
unmarshaller.unmarshal(new File("Item.xml")); // reading from file 
... 
unmarshaller.unmarshal(inputStream); // inputStream is an instance of java.io.InputStream, reading from stream 


Validation with XML Schema 

Last thing I want to mention here is validating input XML with schema before unmarshalling to Java object. I create an XML schema file called item.xsd. For complete code Listing please see Code Listing 4 (Item.xsd). Now what I have to do is register this schema for validation. 

... 
Schema schema = SchemaFactory.newInstance(XMLConstants.W3C_XML_SCHEMA_NS_URI) 
.newSchema(new File("Item.xsd")); 
unmarshaller.setSchema(schema); //register item.xsd shcema for validation 
... 


When I try to unmarshal XML data to POJO, if the input XML is not conformed to the schema, 
exception will be caught. For complete code Listing please see Code Listing 5 (invalid_item.xml). 


javax.xml.bind.UnmarshalException 
- with linked exception: 
javax.xml.bind.JAXBException caught: null 
[org.xml.sax.SAXParseException: cvc-datatype-valid.1.2.1: 'item1' is not a valid value for 'integer'.] 



Here I change the 'id' attribute to string instead of integer. 

If XML input is valid against the schema, the XML data will be unmarshalled to Item.java object successfully. 

I leave out detail on data encryption and XML digital signature and gazillion of things you 
can do with JAXB. My goal here is to make the case for JAXB as an alternative way for 
XML serialization. For more information on JAXB, please check out these links: 

- Kohsuke's blog 
- Netbeans 6 JAXB tutorial 
- Java EE 5 JAXB tutorial 
 
