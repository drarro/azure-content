﻿<properties linkid="dev-java-how-to-on-premise-application-with-blob-storage" urlDisplayName="Image Gallery w/ Storage" pageTitle="On-premises application with blob storage (Java) - Windows Azure" metaKeywords="Azure blob storage, Azure blob Java, Azure blob example, Azure blob tutorial" metaDescription="Learn how to create a console application that uploads an image to Windows Azure, and then displays the image in your browser. Code samples in Java." metaCanonical="" disqusComments="1" umbracoNaviHide="0" writer="waltpo" />


# On-Premises Application with Blob Storage

The following example shows you how you can use Windows Azure storage to
store images in Windows Azure. The code below is for a console
application that uploads an image to Windows Azure, and then creates an
HTML file that displays the image in your browser.

## Table of Contents

-   [Prerequisites][]
-   [To use Windows Azure blob storage to upload a file][]
-   [To delete a container][]

## <a name="bkmk_prerequisites"> </a>Prerequisites

1.  A Java Developer Kit (JDK), v1.6 or later, is installed.
2.  The Windows Azure SDK is installed.
3.  The JAR for the Windows Azure Libraries for Java, and any applicable
    dependency JARs, are installed and are in the build path used by
    your Java compiler. For information on installing the Windows Azure Libraries for Java, see [Download the
    Windows Azure SDK for Java].
4.  A Windows Azure storage account has been set up. The account name
    and account key for the storage account will be used by the code
    below. See [How to Create a Storage Account] for information about creating a storage account,
    and [How to Manage Storage Accounts] for information about retrieving the
    account key.
5.  You have created a local image file named stored at the path
    c:\\myimages\\image1.jpg. Alternatively, modify the
    **FileInputStream** constructor in the example to use a different
    image path and file name.

<div chunk="../../Shared/Chunks/create-account-note.md" />

## <a name="bkmk_uploadfile"> </a>To use Windows Azure blob storage to upload a file

A step-by-step procedure is presented here; if you’d like to skip ahead,
the entire code is presented later in this topic.

Begin the code by including imports for the Windows Azure core storage
classes, the Windows Azure blob client classes, the Java IO classes, and
the **URISyntaxException** class:

    import com.microsoft.windowsazure.services.core.storage.*;
    import com.microsoft.windowsazure.services.blob.client.*;
    import java.io.*;
    import java.net.URISyntaxException;

Declare a class named **StorageSample**, and include the open bracket,
**{**.

    public class StorageSample {

Within the **StorageSample** class, declare a string variable that will
contain the default endpoint protocol, your storage account name, and
your storage access key, as specified in your Windows Azure storage
account. Replace the placeholder values **your\_account\_name** and
**your\_account\_key** with your own account name and account key,
respectively.

    public static final String storageConnectionString = 
           "DefaultEndpointsProtocol=http;" + 
               "AccountName=your_account_name;" + 
               "AccountKey=your_account_name"; 

Add in your declaration for **main**, include a **try** block, and
include the necessary open brackets, **{**.

    public static void main(String[] args) 
    {
        try
        {

Declare variables of the following type (the descriptions are for how
they are used in this example):

-   **CloudStorageAccount**: Used to initialize the account object with
    your Windows Azure storage account name and key, and to create the
    blob client object.
-   **CloudBlobClient**: Used to access the blob service.
-   **CloudBlobContainer**: Used to create a blob container, list the
    blobs in the container, and delete the container.
-   **CloudBlockBlob**: Used to upload a local image file to the
    container.

<!-- -->

    CloudStorageAccount account;
    CloudBlobClient serviceClient;
    CloudBlobContainer container;
    CloudBlockBlob blob;

Assign a value to the **account** variable.

    account = CloudStorageAccount.parse(storageConnectionString);

Assign a value to the **serviceClient** variable.

    serviceClient = account.createCloudBlobClient();

Assign a value to the **container** variable. We’ll get a reference to a
container namedgettingstarted.

    // Container name must be lower case.
    container = serviceClient.getContainerReference("gettingstarted");

Create the container. This method will create the container if doesn’t
exist (and return **true**). If the container does exist, it will return
**false**. An alternative to **createIfNotExist** is the **create**
method (which will return an error if the container already exists).

    container.createIfNotExist();

Set anonymous access for the container.

    // Set anonymous access on the container.
    BlobContainerPermissions containerPermissions;
    containerPermissions = new BlobContainerPermissions();
    containerPermissions.setPublicAccess(BlobContainerPublicAccessType.CONTAINER);
    container.uploadPermissions(containerPermissions);

Get a reference to the block blob, which will represent the blob in
Windows Azure storage.

    blob = container.getBlockBlobReference("image1.jpg");

Use the **File** constructor to get a reference to the locally created
file that you will upload. (Ensure you have created this file before
running the code.)

    File fileReference = new File ("c:\\myimages\\image1.jpg");

Upload the local file through a call to the **CloudBlockBlob.upload**
method. The first parameter to the **CloudBlockBlob.upload** method is a
**FileInputStream** object that represents the local file that will be
uploaded to Windows Azure storage. The second parameter is the size, in
bytes, of the file.

    blob.upload(new FileInputStream(fileReference), fileReference.length());

Call a helper function named **MakeHTMLPage**, to make a basic HTML page
that contains an **&lt;image&gt;** element with the source set to the blob that
is now in your Windows Azure storage account. (The code for
**MakeHTMLPage** will be discussed later in this topic.)

    MakeHTMLPage(container);

Print out a status message and information about the created HTML page.

    System.out.println("Processing complete.");
    System.out.println("Open index.html to see the images stored in your storage account.");

Close the **try** block by inserting a close bracket: **}**

Handle the following exceptions:

-   **FileNotFoundException**: Can be thrown by the **FileInputStream**
    or **FileOutputStream** constructors.
-   **StorageException**: Can be thrown by the Windows Azure client
    storage library.
-   **URISyntaxException**: Can be thrown by the **ListBlobItem.getUri**
    method.
-   **Exception**: Generic exception handling.

<!-- -->

    catch (FileNotFoundException fileNotFoundException)
    {
        System.out.print("FileNotFoundException encountered: ");
        System.out.println(fileNotFoundException.getMessage());
        System.exit(-1);
    }
    catch (StorageException storageException)
    {
        System.out.print("StorageException encountered: ");
        System.out.println(storageException.getMessage());
        System.exit(-1);
    }
    catch (URISyntaxException uriSyntaxException)
    {
        System.out.print("URISyntaxException encountered: ");
        System.out.println(uriSyntaxException.getMessage());
        System.exit(-1);
    }
    catch (Exception e)
    {
        System.out.print("Exception encountered: ");
        System.out.println(e.getMessage());
        System.exit(-1);
    }

Close **main** by inserting a close bracket: **}**

Create a method named **MakeHTMLPage** to create a basic HTML page. This
method has a parameter of type **CloudBlobContainer**, which will be
used to iterate through the list of uploaded blobs. This method will
throw exceptions of type **FileNotFoundException**, which can be thrown
by the **FileOutputStream** constructor, and **URISyntaxException**,
which can be thrown by the **ListBlobItem.getUri** method. Include the
opening bracket, **{**.

    public static void MakeHTMLPage(CloudBlobContainer container) throws FileNotFoundException, URISyntaxException
    {

Create a local file named **index.html**.

    PrintStream stream;
    stream = new PrintStream(new FileOutputStream("index.html"));

Write to the local file, adding in the **&lt;html&gt;**, **&lt;header&gt;**, and
**&lt;body&gt;** elements.

    stream.println("<html>");
    stream.println("<header/>");
    stream.println("<body>");

Iterate through the list of uploaded blobs. For each blob, in the HTML
page create an **&lt;img&gt;** element that has its **src** attribute sent to
the URI of the blob as it exists in your Windows Azure storage account.
Although you added only one image in this sample, if you added more,
this code would iterate all of them.

For simplicity, this example assumes each uploaded blob is an image. If
you’ve updated blobs other than images, or page blobs instead of block
blobs, adjust the code as needed.

    // Enumerate the uploaded blobs.
    for (ListBlobItem blobItem : container.listBlobs()) {
    // List each blob as an <img> element in the HTML body.
    stream.println("<img src='" + blobItem.getUri() + "'/><br/>");
    }

Close the **&lt;body&gt;** element and the **&lt;html&gt;** element.

    stream.println("</body>");
    stream.println("</html>");

Close the local file.

    stream.close();

Close **MakeHTMLPage** by inserting a close bracket: **}**

Close **StorageSample** by inserting a close bracket: **}**

The following is the complete code for this example. Remember to modify
the placeholder values **your\_account\_name** and
**your\_account\_key** to use your account name and account key,
respectively.

    import com.microsoft.windowsazure.services.core.storage.*;
    import com.microsoft.windowsazure.services.blob.client.*;
    import java.io.*;
    import java.net.URISyntaxException;

    // Create an image, c:\myimages\image1.jpg, prior to running this sample.
    // Alternatively, change the value used by the FileInputStream constructor 
    // to use a different image path and file that you have already created.
    public class StorageSample {

        public static final String storageConnectionString = 
                "DefaultEndpointsProtocol=http;" + 
                   "AccountName=your_account_name;" + 
                   "AccountKey=your_account_name"; 

        public static void main(String[] args) 
        {
            try
            {
                CloudStorageAccount account;
                CloudBlobClient serviceClient;
                CloudBlobContainer container;
                CloudBlockBlob blob;
                
                account = CloudStorageAccount.parse(storageConnectionString);
                serviceClient = account.createCloudBlobClient();
                // Container name must be lower case.
                container = serviceClient.getContainerReference("gettingstarted");
                container.createIfNotExist();
                
                // Set anonymous access on the container.
                BlobContainerPermissions containerPermissions;
                containerPermissions = new BlobContainerPermissions();
                containerPermissions.setPublicAccess(BlobContainerPublicAccessType.CONTAINER);
                container.uploadPermissions(containerPermissions);

                // Upload an image file.
                blob = container.getBlockBlobReference("image1.jpg");
                File fileReference = new File ("c:\\myimages\\image1.jpg");
                blob.upload(new FileInputStream(fileReference), fileReference.length());

                // At this point the image is uploaded.
                // Next, create an HTML page that lists all of the uploaded images.
                MakeHTMLPage(container);

                System.out.println("Processing complete.");
                System.out.println("Open index.html to see the images stored in your storage account.");

            }
            catch (FileNotFoundException fileNotFoundException)
            {
                System.out.print("FileNotFoundException encountered: ");
                System.out.println(fileNotFoundException.getMessage());
                System.exit(-1);
            }
            catch (StorageException storageException)
            {
                System.out.print("StorageException encountered: ");
                System.out.println(storageException.getMessage());
                System.exit(-1);
            }
            catch (URISyntaxException uriSyntaxException)
            {
                System.out.print("URISyntaxException encountered: ");
                System.out.println(uriSyntaxException.getMessage());
                System.exit(-1);
            }
            catch (Exception e)
            {
                System.out.print("Exception encountered: ");
                System.out.println(e.getMessage());
                System.exit(-1);
            }
        }

        // Create an HTML page that can be used to display the uploaded images.
        // This example assumes all of the blobs are for images.
        public static void MakeHTMLPage(CloudBlobContainer container) throws FileNotFoundException, URISyntaxException
    {
            PrintStream stream;
            stream = new PrintStream(new FileOutputStream("index.html"));

            // Create the opening <html>, <header>, and <body> elements.
            stream.println("<html>");
            stream.println("<header/>");
            stream.println("<body>");

            // Enumerate the uploaded blobs.
            for (ListBlobItem blobItem : container.listBlobs()) {
                // List each blob as an <img> element in the HTML body.
                stream.println("<img src='" + blobItem.getUri() + "'/><br/>");
            }

            stream.println("</body>");

            // Complete the <html> element and close the file.
            stream.println("</html>");
            stream.close();
        }
    }

In addition to uploading your local image file to Windows Azure storage,
the example code creates a local file namedindex.html, which you can
open in your browser to see your uploaded image.

Because the code contains your account name and account key, ensure that
your source code is secure.

## <a name="bkmk_deletecontainer"> </a>To delete a container

Because you are charged for storage, you may want to delete the
gettingstartedcontainer after you are done experimenting with this
example. To delete a container, use the **CloudBlobContainer.delete**
method:

    container = serviceClient.getContainerReference("gettingstarted");
    container.delete();

To call the **CloudBlobContainer.delete** method, the process of
initializing **CloudStorageAccount**, **ClodBlobClient**,
**CloudBlobContainer** objects is the same as shown for the
**createIfNotExist** method. The following is a complete example that
deletes the container namedgettingstarted.

    import com.microsoft.windowsazure.services.core.storage.*;
    import com.microsoft.windowsazure.services.blob.client.*;

    public class DeleteContainer {

        public static final String storageConnectionString = 
                "DefaultEndpointsProtocol=http;" + 
                   "AccountName=your_account_name;" + 
                   "AccountKey=your_account_key"; 

        public static void main(String[] args) 
        {
            try
            {
                CloudStorageAccount account;
                CloudBlobClient serviceClient;
                CloudBlobContainer container;
                
                account = CloudStorageAccount.parse(storageConnectionString);
                serviceClient = account.createCloudBlobClient();
                // Container name must be lower case.
                container = serviceClient.getContainerReference("gettingstarted");
                container.delete();
                
                System.out.println("Container deleted.");

            }
            catch (StorageException storageException)
            {
                System.out.print("StorageException encountered: ");
                System.out.println(storageException.getMessage());
                System.exit(-1);
            }
            catch (Exception e)
            {
                System.out.print("Exception encountered: ");
                System.out.println(e.getMessage());
                System.exit(-1);
            }
        }
    }

For an overview of other blob storage classes and methods, see [How to
Use the Blob Storage Service from Java].

  [Prerequisites]: #bkmk_prerequisites
  [To use Windows Azure blob storage to upload a file]: #bkmk_uploadfile
  [To delete a container]: #bkmk_deletecontainer
  [Download the Windows Azure SDK for Java]: http://www.windowsazure.com/en-us/develop/java/
  [How to Create a Storage Account]: http://www.windowsazure.com/en-us/manage/services/storage/how-to-create-a-storage-account/
  [How to Manage Storage Accounts]: http://www.windowsazure.com/en-us/manage/services/storage/how-to-manage-a-storage-account/
  [How to Use the Blob Storage Service from Java]: http://www.windowsazure.com/en-us/develop/java/how-to-guides/blob-storage/
