When installing a Chocolatey Package, it is possible to use a number of arguments to control how the package is installed.  Each one of these arguments is detailed [here](https://github.com/chocolatey/chocolatey/wiki/CommandsInstall).  

This _How-To_ focuses on how a package creator can make use of the PackageParameters argument within their package, and how they can parse the string which is passed through into their package from the installation command.

## Code Sample

```powershell

$arguments = @{};

# Let's assume that the input string is something like this, and we will use a Regular Expression to parse the values
# /Port:81 /Edition:LicenseKey

# Now, we can use the $env:chocolateyPackageParameters inside the Chocolatey package
$packageParameters = $env:chocolateyPackageParameters;

# Default the values
$port = "81";
$edition = "LicenseKey";

# Now, let’s parse the packageParameters using good old regular expression
if($packageParameters) {
    $MATCH_PATTERN = "/([a-zA-Z]+):([`"'])?([a-zA-Z0-9- _]+)([`"'])?"
    $PARAMATER_NAME_INDEX = 1
    $VALUE_INDEX = 3
    
    if($packageParameters -match $MATCH_PATTERN ){
        $results = $packageParameters | Select-String $MATCH_PATTERN -AllMatches 
        $results.matches | % { 
          $arguments.Add(
              $_.Groups[$PARAMATER_NAME_INDEX].Value.Trim(),
              $_.Groups[$VALUE_INDEX].Value.Trim()) 
      }
    }     
    
    if($arguments.ContainsKey("Port")) {
        Write-Host "Port Argument Found";
        $port = $arguments["Port"];
    }  
    
    if($arguments.ContainsKey("Edition")) {
        Write-Host "Edition Argument Found";
        $edition = $arguments["Edition"];
    }
} else {
    Write-Host "No Package Parameters Passed in";
}

$silentArgs = "/S /Port=" + $port + " /Edition=" + $edition;

Write-Host "This would be the Chocolatey Silent Arguments: $silentArgs"
```

## What does this mean?

The Code Sample assumes that there will be no PackageParameters passed into it, as a result, we need to define a number of default values for each of the variables contained within the script.  In this case, the ```port``` and the ```edition```.

Once that is done, assuming that the PackageParameters contains "something", use a Regular Expression to parse each of the values into a dictionary.  Here, we are assuming that the package parameters will come through in a pre-defined format, such as ```/Port:81 /Edition:LicenseKey```.  Now, this format can be anything you want it to be.  What is shown here is just **one** way of doing it.

Having collected all the arguments into the dictionary, we can then inspect the values of each parameter that we are interested in.  If it exists in the dictionary, replace the corresponding default value, otherwise, continue to use the default value.

## Installing the Package
Now, in this example, if we were to call:

```choco install <packageName>```

The output would be:

```
This would be the Chocolatey Silent Arguments: /S /Port=81 /Edition=LicenseKey
```

i.e. it is using the default values which we made at the top of the file

However, if we instead used:

```
choco install <packageName> -packageParameters "/Port:82 /Edition=LicenseKey1"
```

The output would be:

```
This would be the Chocolatey Silent Arguments: /S /Port=82 /Edition=LicenseKey1
```