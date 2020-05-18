# Items vs Objects

The difference between items and objects in PowerShell can seem slight to the new user. This is an attempt to make it more clear.

## Where Does It Live?

This is the true line between the two.

Items exist in non-volatile memory, and are "real" things. Files and directories are most common, but also certificates and the registry. From a Unix/Linux perspective, if it is on a disk it is a file. In Powershell if it is on a disk, it is an item.

Objects exist in volatile memory, and can be an interpretation of an item, or something abstract. You can create an object that describes an item, or an object that describes information only. Objects need to be translated into a form to become items, but that item then does not truly represent that data it contains... More on that later.

## Items

I have created an example directory. Lets see what happens when we run a command against it:

```
PS a:\> Get-ChildItem .\exampleDirectory\


    Directory: a:\exampleDirectory


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        5/12/2020   6:33 PM         238080 bemo.gif
-a----        5/12/2020   6:33 PM          14812 powershell.png
-a----        5/17/2020   7:46 AM            113 someText.txt
```

``Get-ChildItem`` is similar to ``DIR`` in CMD or ``ls -l`` in bash. For example, in gitbash:

```
$ ls -l
total 253
-rw-r--r-- 1 David 197609 238080 May 12 18:33 bemo.gif
-rw-r--r-- 1 David 197609  14812 May 12 18:33 powershell.png
-rw-r--r-- 1 David 197609    113 May 17 07:46 someText.txt
```

In CMD it looks pretty much like ``Get-ChildItem``. So what does that mean? Mainly that an item is something that has properties regardless or powershell. ``bemo.gif`` is 238080 bytes regardless of operating system or shell. Bytes are bytes, and that's that. Also, the filename does not change, nor do many other aspects.

## Objects

Objects are collections or properties. Everything in PowerShell is an object.

### Objects Describing Items

PowerShell describes items with objects. Let's look at ``bemo.gif`` again. Above we see that ``Get-ChildItem`` shows us Name, Length (bytes), Last Write Time, and Mode; but what else is there?

Try this:

```powershell
Get-ChildItem .\exampleDirectory\bemo.gif | Select-Object -Property *
```

``Get-ChildItem`` retrieves the item's characteristics and makes an object of it. The item's characteristics are represented in the object as properties. Powershell adds some of its own properties to assist in how it treats the item.

```
PSPath            : Microsoft.PowerShell.Core\FileSystem::a:\exampleDirectory\bemo.gif
PSParentPath      : Microsoft.PowerShell.Core\FileSystem::a:\exampleDirectory
PSChildName       : bemo.gif
PSDrive           : a
PSProvider        : Microsoft.PowerShell.Core\FileSystem
PSIsContainer     : False
Mode              : -a----
VersionInfo       : File:             a:\exampleDirectory\bemo.gif
                    InternalName:
                    OriginalFilename:
                    FileVersion:
                    FileDescription:
                    Product:
                    ProductVersion:
                    Debug:            False
                    Patched:          False
                    PreRelease:       False
                    PrivateBuild:     False
                    SpecialBuild:     False
                    Language:

BaseName          : bemo
Target            : {a:\exampleDirectory\bemo.gif}
LinkType          :
Name              : bemo.gif
Length            : 238080
DirectoryName     : a:\exampleDirectory
Directory         : a:\exampleDirectory
IsReadOnly        : False
Exists            : True
FullName          : a:\exampleDirectory\bemo.gif
Extension         : .gif
CreationTime      : 5/17/2020 7:49:22 AM
CreationTimeUtc   : 5/17/2020 11:49:22 AM
LastAccessTime    : 5/17/2020 7:49:22 AM
LastAccessTimeUtc : 5/17/2020 11:49:22 AM
LastWriteTime     : 5/12/2020 6:33:26 PM
LastWriteTimeUtc  : 5/12/2020 10:33:26 PM
Attributes        : Archive
```

Anything above that begins with ``PS`` is a PowerShell specific property, some are interpretation properties, and others are item characteristics.

Some of the interpreted properties are things like ``Extension``, which is just ``.gif`` above, but what if we copy ``bemo.gif`` to ``bemo.gif.bak``?

```
Get-ChildItem .\bemo.gif.bak | Select-Object -Property Name,BaseName,Extension | Format-List

Name      : bemo.gif.bak
BaseName  : bemo.gif
Extension : .bak
```

Just to show that these are still the same files, let's look at their checksum:

```
Get-FileHash -Algorithm SHA1 -Path bemo* | Select-Object -Property Path,Hash | Format-List

Path : a:\exampleDirectory\bemo.gif
Hash : 7D6F6F25B3D8D27DB9B7AB81EC41E240577E5A7A

Path : a:\exampleDirectory\bemo.gif.bak
Hash : 7D6F6F25B3D8D27DB9B7AB81EC41E240577E5A7A
```

### Abstract Objects

You can make your own objects in Powershell, simply by assigning a variable a value:

```
$MyVar = "PowerShell is cool, right?"
$MyVar.GetType()

IsPublic IsSerial Name     BaseType
-------- -------- ----     --------
True     True     String   System.Object
```

``$MyVar`` is just a string, but its BaseType is ``System.Object``.

```
$MyVar = 1
$MyVar.GetType()

IsPublic IsSerial Name     BaseType
-------- -------- ----     --------
True     True     Int32    System.ValueType
```

Now ``$MyVar`` is an integer, but still its BaseType is ``System.Object``.

Things get interesting when you start working with complex objects, like a hash table.

```powershell
$MyVar = @{
    Name = "David"
    Car = "Fiat 500 Pop"
}
```

If you know any javascript or PHP this might look a lot like an associative array, and it is, but in PowerShell it is a hash table.

```
$MyVar

Name                           Value
----                           -----
Car                            Fiat 500 Pop
Name                           David

$MyVar.GetType()

IsPublic IsSerial Name         BaseType
-------- -------- ----         --------
True     True     Hashtable    System.Object
```

## Hybrib Objects

So, earlier, we looked at ``bemo.gif``, and its properties. We could assign the response to a variable, and then get individual properties later. Note that when we do this, the object we create will represent ``bemo.gif`` at the time we created it. Modification of the file (including deletion) will not affect out variable.

What if we only want to know a few things about ``bemo.gif``? We could do:

```
Get-Item .\bemo.gif | Select-Object -Property `
    Fullname,BaseName,Extension,LastWriteTime | Format-List

FullName      : a:\exampleDirectory\bemo.gif
BaseName      : bemo
Extension     : .gif
LastWriteTime : 5/12/2020 6:33:26 PM
```

We also have seen how to get the checksum of ``bemo.gif``:

```
Get-FileHash .\bemo.gif -Algorithm SHA1 | Select-Object -Property `
    Path,Hash,Algorithm | Format-List

Path      : a:\exampleDirectory\bemo.gif
Hash      : 7D6F6F25B3D8D27DB9B7AB81EC41E240577E5A7A
Algorithm : SHA1
```

Let's mix those together:

```
$MyVar = Get-Item .\bemo.gif | Select-Object -Property `
    Fullname,BaseName,Extension,LastWriteTime,Hash,Algorith
$MyVar

FullName      : a:\exampleDirectory\bemo.gif
BaseName      : bemo
Extension     : .gif
LastWriteTime : 5/12/2020 6:33:26 PM
Hash          :
Algorith      :


```








