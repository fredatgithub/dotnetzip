# dotnetzip #
This is a git mirror of DotNetZip, a ZIP file handling library for .NET and
Mono.

You can reach DotNetZip's website at 

This project is licensed under the Ms-PL, packaged within this project as
"License.txt".

-Ed Ropple <ed+dotnetzip@edropple.com>

Create a Zip archive, add files to it, save it. This is the basic programming model for the ZipFile object. Key points of interest, the Using clause is important. The ZipFile object implements the IDisposable interface, indicating that users of the object need to call Dispose() on it when finished using it. The using clause guarantees that the Dispose() method is called implicitly. This will close the file pointer and allow you to move the resulting zip file, rename it, delete it, whatever. If you do not call Dispose(), the generated zip file won't be available to use elsewhere, possibly until your application exits. Also, you won't be able to delete the file, within your own application. Normally, you would surround the code that uses DotNetZip in a try...catch, which is not shown here.

  using (ZipFile zip = new ZipFile())
  {
    zip.AddFile("ReadMe.txt");
    zip.AddFile("7440-N49th.png");
    zip.AddFile("2008_Annual_Report.pdf");        
    zip.Save("Archive.zip");
  }


Unpack or extract items from a zip file. This is probably the 2nd most common thing people do with DotNetZip. Again, note the Using clause. Very important. The key method demonstrated here is the ZipEntry.Extract. The first parameter is the Directory to which to extract. The 2nd parameter tells the Extract() method what to do when an extraction would overwrite an existing file. The options, given by the ExtractExistingFileAction enumeration, are: overwrite silently, throw an exception, don't overwrite (silently), or call into a delegate. This example shows how to overwrite existing files.

  private void MyExtract()
  {
      string zipToUnpack = "C1P3SML.zip";
      string unpackDirectory = "Extracted Files";
      using (ZipFile zip1 = ZipFile.Read(zipToUnpack))
      {
          // here, we extract every entry, but we could extract conditionally
          // based on entry name, size, date, checkbox status, etc.  
          foreach (ZipEntry e in zip1)
          {
            e.Extract(unpackDirectory, ExtractExistingFileAction.OverwriteSilently);
          }
       }
    }



Create a Zip archive, and write it to an output stream. To save a zip file to a stream, you use the same basic programming model, but call Save() and pass a writable output stream.

  using (ZipFile zip = new ZipFile())
  {
    zip.AddFile("ReadMe.txt");
    zip.AddFile("7440-N49th.png");
    zip.AddFile("2008_Annual_Report.pdf");        
    zip.Save(outputStream);
  }



Add a set of items to a zip file. This example adds entries to a zip file. Each entry is added using the pathname in the name of the file. Readme will appear in the root of the zip archive, the .csv file will appear in the data directory, and the .pdf will appear in the reports directory.

  String[] filenames = { "ReadMe.txt", "c:\\data\\collection.csv", "c:\\reports\\AnnualSummary.pdf"};
  using (ZipFile zip = new ZipFile())
  {
    zip.AddFiles(filenames);
    zip.Save("Archive.zip");
  }


Add a set of items to a zip file, specifying a common directory in the zip archive. This example adds entries to a zip file. Each entry is added using the specified pathname.

  String[] filenames = { "ReadMe.txt", "c:\\data\\collection.csv", "c:\\reports\\AnnualSummary.pdf"};
  using (ZipFile zip = new ZipFile())
  {
    zip.AddFiles(filenames, "files");
    zip.Save("Archive.zip");
  }


Add an entry with a comment. This example zips up a single file, and adds a comment to it. The zip specification allows a comment to be added to each entry.

  using (ZipFile zip = new ZipFile())
  {
    ZipEntry e= zip.AddFile("BusinessPlan-2a.xslx");
    e.Comment = "This is the version I worked out with Chris.";
    zip.Save("Archive.zip");
  }


Add an entry with a comment. Same as above with a more compact syntax

  using (ZipFile zip = new ZipFile())
  {
    zip.AddFile("BusinessPlan-2a.xslx").Comment = "This is the version I worked out with Chris.";
    zip.Save("Archive.zip");
  }


Add an entry, overriding its name in the archive. This is how you can zip up a file, and rename it within the archive . Upon extraction, a file will be created with the new name.

  using (ZipFile zip1 = new ZipFile())
  {
      string newName= fileToZip + "-renamed";
      zip1.AddFile(fileToZip).FileName = newName; 
      zip1.Save(archiveName);
  }


Create a zip archive with password protection. This is the same as the first example, but we've added password protection into the mix. The code creates a new ZipFile instance, and then specifies a password for it. It then adds a few files into the zip archive. The password is implicitly applied to the entries added to the ZipFile. Upon extraction in WinZip, Windows Explorer, or some other tool, the user will have to specify the appropriate password to extract the given files. Keep in mind that using password protection in Zip files does not protect the list of files contained in the Zip archive, nor the details of those files, like filename, size, date, and so on. Finally, once again notice the ever-present using clause.

using (ZipFile zip = new ZipFile())
{
    zip.Password= "123456!";
    zip.AddFile("ReadMe.txt");
    zip.AddFile("7440-N49th.png");
    zip.AddFile("2005_Annual_Report.pdf");        
    zip.Save("Backup.zip");
}


Create a zip using Password protection, different passwords for each entry. Add items to a zip file, using different passwords for different items, and a comment on one of the items.

  using (ZipFile zip = new ZipFile())
  {
    zip.AddFile("ReadMe.txt"); // no password for this entry

    zip.Password= "123456!";
    ZipEntry e = zip.AddFile("7440-N49th.png");
    e.Comment = "Map of the company headquarters."; 

    zip.Password= "!Secret1";
    zip.AddFile("2Q2008_Operations_Report.pdf");
    
    zip.Save("Backup.zip");
  }


Create a zip using Password protection, different passwords for each entry. This example uses an alternative syntax to the above.

  using (ZipFile zip = new ZipFile())
  {
    zip.AddFile("ReadMe.txt"); // no password for this entry

    ZipEntry e = zip.AddFile("7440-N49th.png");
    e.Comment = "Map of the company headquarters."; 
    e.Password= "123456!";

    zip.AddFile("2Q2008_Operations_Report.pdf").Password = "!Secret1";
    
    zip.Save("Backup.zip");
  }


Create a Zip archive that uses WinZip-compatible AES encryption. Answering concerns that the standard password protection supported by all zip tools is weak, WinZip has extended the ZIP specification and added a way to use AES Encryption to protect entries in the Zip file. Unlike the "PKZIP encryption" specified in the PKZIP spec, AES Encryption (http://en.wikipedia.org/wiki/Advanced_Encryption_Standard) is a general encryption standard, usable to encrypt any sort of data, not just zip entries. Along with being general, it is also a much stronger encryption algorithm that the PKZIP approach - harder to crack without the password. And, AES can use variable key sizes, to vary the strength even more. WinZip supports 128-bit and 256-bit AES Encryption. DotNetZip supports the same file format as WinZip, therefore, creating a zip file that employs WinZip AES encryption using DotNetZip will result in zip archive that is readable by WinZip. It all sounds very complicated but it is very easy to use with DotNetZip - all the details are taken care of. Keep in mind though, that Zip archives that use WinZip-compatible AES encryption may not be extractable using tools other than WinZip. For example, Windows Explorer cannot unpack a "compressed folder" that uses AES encryption. But, if you really want your ZipEntries to be private, you should use AES encryption.

  using (ZipFile zip = new ZipFile())
  {
    zip.AddFile("ReadMe.txt"); // unencrypted
    zip.Password= "123456!";
    zip.Encryption = EncryptionAlgorithm.WinZipAes256;
    zip.AddFile("7440-N49th.png");
    zip.AddFile("2008_Annual_Report.pdf");        
    zip.Save("Backup.zip");
  }


Create a zip without compression. Create a zip archive, but tell the library to not use compression when adding in the file. This might make sense with previously-compressed files like those in .mp3 format. A further compression of these files would gain nothing in terms of space, but would incur CPU overhead:

  using (ZipFile zip = new ZipFile())
  {
    zip.CompressionLevel = Ionic.Zlib.CompressionLevel.None;
    zip.AddFile(@"MyMusic\Messiah-01.mp3");
    zip.Save(ZipFileToCreate);
  }


Create a zip and specify the compression level. This example creates a zip, and specifies the compression level to use for the entry in the archive. Applications can set the compression level to adjust the balance between speed and efficiency of the compression.

  using (ZipFile zip = new ZipFile())
  {
    zip.CompressionLevel= Ionic.Zlib.CompressionLevel.BestCompression;
    zip.AddFile("Datafeed-2008-12-20.csv");
    zip.Save(ZipFileToCreate);
  }


Create a zip containing all the files in a folder. Zip up an entire directory, recursively, and specify a comment on the zip archive when creating it:

  using (ZipFile zip = new ZipFile())
  {
    zip.AddDirectory(@"MyDocuments\ProjectX", "ProjectX");
    zip.Comment = "This zip was created at " + System.DateTime.Now.ToString("G") ; 
    zip.Save(zipFileToCreate);
  }


Create a zip containing all the files in a folder. This is an alternative to the above.

  using (ZipFile zip = new ZipFile())
  {
    string[] files = Directory.GetFiles(@"MyDocuments\ProjectX");
    // add all those files to the ProjectX folder in the zip file
    zip.AddFiles(files, "ProjectX"); 
    zip.Comment = "This zip was created at " + System.DateTime.Now.ToString("G") ; 
    zip.Save(zipFileToCreate);
  }


Create a split zip containing all the files in a folder. This is like the above, except it will produce multiple files in output, each limited to 2mb in size. The files will be named Projext.z01, Projext.z02, Projext.z03, ... Projext.zip. The resulting zip file can be opened by WinZip, PKZip, DotNetZip, or any tool or library that supports split or "spanned" zips.

  using (ZipFile zip = new ZipFile())
  {
    string[] files = Directory.GetFiles(@"MyDocuments\ProjectX");
    // add all those files to the ProjectX folder in the zip file
    zip.AddFiles(files, "ProjectX"); 
    zip.Comment = "This zip was created at " + System.DateTime.Now.ToString("G") ; 
    zip.MaxOutputSegmentSize = 2*1024*1024;   // 2mb
    zip.Save("ProjextX.zip");
  }


Set the timestamp on each entry as it is added. This example zips up a set of files, each protected by the same password. Also, it overrides the last modified time for all of the files as they are stored in the zip archive:

  using (ZipFile zip = new ZipFile())
  {
    zip1.Password = Password;
    System.DateTime Timestamp = new DateTime(DateTime.Now.Year,DateTime.Now.Month,DateTime.Now.Day,12,0,0);
    String[] filenames = System.IO.Directory.GetFiles("ProjectDocuments");
    foreach (String f in filenames)
    {
      ZipEntry e = zip.AddFile(f, "docs");
      e.LastModified = Timestamp;
    }
    zip.Comment = "This zip was created at " + System.DateTime.Now.ToString("G") ; 
    zip.Save(ZipFileToCreate);
  }


Remap directories. Zip up a set of files and directories, and re-map them into a different directory hierarchy in the zip file:

  using (ZipFile zip = new ZipFile())
  {
    // files in the filesystem like MyDocuments\ProjectX\File1.txt , will be stored in the zip archive as  backup\File1.txt
    zip.AddDirectory(@"MyDocuments\ProjectX", "backup");
    // files in the filesystem like MyMusic\Santana\OyeComoVa.mp3, will be stored in the zip archive as  tunes\Santana\OyeComoVa.mp3
    zip.AddDirectory("MyMusic", "tunes");
    // The Readme.txt file in the filesystem will be stored in the zip archive as documents\Readme.txt
    zip.AddDirectory("Readme.txt", "documents");

    zip.Comment = "This zip was created at " + System.DateTime.Now.ToString("G") ; 
    zip.Save(ZipFileToCreate);
  }


Use wildcards. Zip up a set of files using wildcards to select files from the current working directory (first available in v1.8):

  using (ZipFile zip = new ZipFile())
  {
    zip.AddSelectedFiles("*.docx");
    zip.Save(ZipFileToCreate);
  }


Zip up using wildcards and subdirectories. Zip up a set of files using wildcards to select files from the current working directory, and recurse through subdirectories (first available in v1.8):

  bool recurseDirectories = true;
  using (ZipFile zip = new ZipFile())
  {
    zip.AddSelectedFiles("*.docx", recurseDirectories);
    zip.Save(ZipFileToCreate);
  }


Zip up a particular folder using wildcards and subdirectories. Another variation of the above; this one specifies the directory on disk to search in.

  bool recurseDirectories = true;
  using (ZipFile zip = new ZipFile())
  {
    zip.AddSelectedFiles("*.docx", @"C:\MyDocuments", recurseDirectories);
    zip.Save(ZipFileToCreate);
  }


Zip up using a selector string. Another variation of the above; this one zips up docx files modified since June 30th.

  bool recurseDirectories = true;
  string selection = "(name = *.docx) AND (mtime > 2009-06-30)";
  using (ZipFile zip = new ZipFile())
  {
    zip.AddSelectedFiles(selection, @"C:\MyDocuments", recurseDirectories);
    zip.Save(ZipFileToCreate);
  }




Create a zip using content obtained from a stream. Add an entry into an archive, taking content for the entry from a stream. Also, add a comment to the entry that was added from the stream:

  using (ZipFile zip = new ZipFile())
  {
    ZipEntry e= zip.AddEntry("Content-From-Stream.bin", "basedirectory", StreamToRead);
    e.Comment = "The content for entry in the zip file was obtained from a stream";
    zip.AddFile("Readme.txt");
    zip.Save(zipFileToCreate);
  }



Update a zip: Remove an entry. Remove an entry from an existing zip file, using the string indexer:

  using (ZipFile zip = ZipFile.Read(ExistingZipFile))
  {
    // use the indexer to remove the file from the zip archive
    zip["Readme.txt"] = null;
    zip.Comment = "This archive has been modified from its original version. Some files have been removed.";
    zip.Save();
  }



Update a zip: Remove an entry, alternative. Remove an entry from an existing zip file, using the RemoveEntry method.

  using (ZipFile zip = ZipFile.Read(ExistingZipFile))
  {
    zip.RemoveEntry("Readme.txt");
    zip.Comment = "This archive has been modified from its original version. Some files have been removed.";
    zip.Save();
  }




Extract entries. Extract all files from a zip archive. This example works if you want to extract all entries, and none of them use a password for encryption. Within the foreach loop, you could insert a conditional to determine if you want to extract that particular entry. You could query the timestamp on the entry, the size, the file attributes, and so on.

  using (ZipFile zip = ZipFile.Read(ExistingZipFile))
  {
    foreach (ZipEntry e in zip)
    {
      e.Extract(TargetDirectory);
    }
  }



Extract entries, an alternative. Extract all files from a zip archive. This example works if you want to extract all entries, and none of them use a password for encryption.

  using (ZipFile zip = ZipFile.Read(ExistingZipFile))
  {
      zip.ExtractAll(TargetDirectory);    
  }



Extract with overwrite. The default behavior of extraction is to NOT overwrite existing files. If there are files that would be overwritten, DotNetZip throws an exception. In this example, the app Extracts all files, and silently overwrites any existing files in the filesystem. There are alternatives not shown here, to "overwrite silently": Throw an exception, don't overwrite, or invoke an event to allow the application to decide what to do.

  using (ZipFile zip = ZipFile.Read(ExistingZipFile))
  {
    foreach (ZipEntry e in zip)
    {
      e.Extract(TargetDirectory, ExtractExistingFileAction.OverwriteSilently);
    }
  }



Extract into a stream. Extract an entry from the zip archive into a previously-opened stream:

  using (ZipFile zip = ZipFile.Read(ExistingZipFile))
  {
    ZipEntry e = zip["MyReport.doc"];
    e.Extract(OutputStream);
  }



Extract an encrypted entry. Extract an Entry that was encrypted with a password, into the specified base directory:

  using (ZipFile zip = ZipFile.Read(ExistingZipFile))
  {
    ZipEntry e = zip["TaxInformation-2008.xls"];
    e.ExtractWithPassword(BaseDirectory, Password);
  }



Extract an Entry that was encrypted with a password, an alternative.

  using (ZipFile zip = ZipFile.Read(ExistingZipFile))
  {
    ZipEntry e = zip["TaxInformation-2008.xls"];
    e.Password = Password;
    e.Extract(BaseDirectory);
  }



List entries in a zip. List all the entries in a zip file:

  using (ZipFile zip = ZipFile.Read(ExistingZipFile))
  {
    foreach (ZipEntry e in zip)
    {
      if (header)
      {
        System.Console.WriteLine("Zipfile: {0}", zip.Name);
        if ((zip.Comment != null) && (zip.Comment != "")) 
          System.Console.WriteLine("Comment: {0}", zip.Comment);
        System.Console.WriteLine("\n{1,-22} {2,8}  {3,5}   {4,8}  {5,3} {0}",
                                 "Filename", "Modified", "Size", "Ratio", "Packed", "pw?");
        System.Console.WriteLine(new System.String('-', 72));
        header = false;
      }
      System.Console.WriteLine("{1,-22} {2,8} {3,5:F0}%   {4,8}  {5,3} {0}",
                               e.FileName,
                               e.LastModified.ToString("yyyy-MM-dd HH:mm:ss"),
                               e.UncompressedSize,
                               e.CompressionRatio,
                               e.CompressedSize,
                               (e.UsesEncryption) ? "Y" : "N");

    }
  }



Input from a stream. This example reads in zip archive content from an input stream, then extracts the content for one entry to a filesysten file. In this example, the filename "NameOfEntryInArchive.doc", refers only to the name of the entry within the zip archive. This name is used as the index in the string indexer on the ZipFile object. The return value is a ZipEntry. The ZipEntry.Extract() method is then called, which extracts the named entry to a filesystem file, using the current working directory as the base. A file by that name is created in the filesystem.

  using (ZipFile zip = ZipFile.Read(InputStream))
  {
    ZipEntry entry = zip["NameOfEntryInArchive.doc"];
    entry.Extract();  // create filesystem file here. 
  }




Extract to a Stream. This example reads a zip archive from a filesystem file, then extracts one of the entries in the archive into a MemoryStream, which is a Stream layered atop a byte array. DotNetZip can extract into any writeable stream. For example, you could extract into a FileStream, if you like, or an FtpStream, or an Http response stream.

  using (var ms = new MemoryStream())
  {
    using (ZipFile zip = ZipFile.Read("MyArchive.zip"))
    {
      ZipEntry entry = zip["NameOfEntryInArchive.doc"];
      entry.Extract(ms);  // extract uncompressed content into a memorystream 
    }   
   // the application can now access the MemoryStream here
  }




Obtain a Readable Stream for extraction. This example reads in a zip archive, then opens a readable stream for one of the entries in the archive. The OpenReader() method is an alternative to the ZipEntry.Extract() overload that accepts a stream. In the former case, the application receives a readable stream, and can obtain decompressed data for the entry from that stream. In the latter case, the application provides a writeable stream, and DotNetZip writes decompressed data into that stream.

  using (ZipFile zip = ZipFile.Read("MyArchive.zip"))
  {
    ZipEntry entry = zip["NameOfEntryInArchive.doc"];
    using (var stream = entry.OpenReader()) 
    {
        var buffer = new byte[2048];
        int n;
        while ((n = stream.Read(buffer, 0, buffer.Length)) > 0) 
        {
            // do something with the buffer here.
        }
    }
  }




Create a downloadable zip within ASP.NET. This example creates a zip dynamically within an ASP.NET postback method, then downloads that zipfile to the requesting browser through Response.OutputStream. No zip archive is ever created on disk.

public void btnGo_Click (Object sender, EventArgs e)
{
  Response.Clear();
  Response.BufferOutput= false;  // for large files
  String ReadmeText= "This is a zip file dynamically generated at " + System.DateTime.Now.ToString("G");
  string filename = System.IO.Path.GetFileName(ListOfFiles.SelectedItem.Text) + ".zip";
  Response.ContentType = "application/zip";
  Response.AddHeader("content-disposition", "filename=" + filename);
  
  using (ZipFile zip = new ZipFile()) 
  {
    zip.AddFile(ListOfFiles.SelectedItem.Text, "files");
    zip.AddEntry("Readme.txt", "", ReadmeText);
    zip.Save(Response.OutputStream);
  }
  Response.Close();
}



Add an entry, getting content from a string. Create a zip with a single entry, obtaining the content for that entry from a string.

  string content = "......whatever....";  
  using (ZipFile zip = new ZipFile())
  {
    zip.AddEntry("README.txt", content);
    zip.Save("archive-2008july12.zip");
  }



Update a zip archive. Open an existing zip archive and modify it: update one entry, remove a second one, and rename a third. Then save.

  using (ZipFile zip = ZipFile.Read("ExistingArchive.zip"))
  {
    // update
    zip.UpdateItem("Portfolio.doc"); 
    // remove
    zip["OldData.txt"].RemoveEntry();
    // rename:
    zip["Internationalization.doc"].FileName = "i18n.doc";
    // add a comment to the archive
    zip.Comment = "This zip archive was updated " + System.DateTime.ToString("G"); 
    zip.Save();
  }




Create a self-extracting archive. This example shows how to create a zip that is a self-extracting archive.

string DirectoryPath = "c:\\Documents\\Project7";
using (ZipFile zip = new ZipFile())
{
    zip.AddDirectory(DirectoryPath, System.IO.Path.GetFileName(DirectoryPath));
    zip.Comment = "This will be embedded into a self-extracting console-based exe";
    SelfExtractorOptions options = new SelfExtractorOptions();
    options.Flavor = SelfExtractorFlavor.ConsoleApplication;
    options.DefaultExtractDirectory = "%USERPROFILE%\\ExtractHere";
    options.PostExtractCommandLine = ExeToRunAfterExtract;
    options.RemoveUnpackedFilesAfterExecute = true;
    zip.SaveSelfExtractor("archive.exe", options);
}




Create a self-extracting archive (ex 2). This one shows how to set the title of the Window that the SFX will run in.

var sfxFileToCreate = "DNZ1-SFX.exe";
using (var zip = new ZipFile())
{
    var filesToAdd = System.IO.Directory.GetFiles(".", "*.cs");
    zip.AddFiles(filesToAdd, "");
    var sfxOptions = new SelfExtractorSaveOptions
    {
        Flavor = SelfExtractorFlavor.WinFormsApplication,
        Quiet = false,
        Copyright = "(c) 2011 Dino P. Chiesa",
        Description = "This is a test",
        SfxExeWindowTitle = "My SFX Window title" // new feature in v1.9.1.6
    };
    zip.SaveSelfExtractor(sfxFileToCreate, sfxOptions);
}




Extract with Progress events: This can be used in a WinForms or WPF app to extract asynchronously.

    public void ExtractFile(string zipToUnpack, string unpackDirectory)
    {
        BackgroundWorker worker = new BackgroundWorker();
        worker.WorkerReportsProgress = true;
        worker.ProgressChanged += (o, e) { .... };
        worker.DoWork += (o, e) =>
        {
            using (ZipFile zip = ZipFile.Read(zipToUnpack))
            {  
                int step = (zip.Count / 100.0);
                int percentComplete = 0;
                foreach (ZipEntry file in zip)
                {
                    file.Extract(unpackDirectory, ExtractExistingFileAction.OverwriteSilently);
                    percentComplete += step;
                    worker.ReportProgress(percentComplete);
                }
            }
        };
    
        worker.RunWorkerAsync();
    }

