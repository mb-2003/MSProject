# MSProject

[Project Introduction Here]

* [Installation](#installation) 
* [Functionalities](#functionalities)
* [Key Definitions](#key-definitions)
* [Other Features](#other-feautures)

# Installation 
## Dependencies

NumPy, ortools, and PubChemPy, along with multiple default Python packages are all needed for full functionality. They can be installed using the following pip requirements file in CLI:

~~~
pip install -r reqs.txt
~~~

# Functionalities
1. [Thermo RAW file Conversion](#1-raw-file-conversion-supported-on-windows-only)

2. [Peak Extraction](#2-peak-extraction)

3. [Peak Processing](#3-peak-processing)

   * [Relative Intensities](#3a-relative-intensities)
 
   * [Averaging Spectra](#3b-averaging)
 
   * [Comparison](#3c-comparison)
 
4. [Potential Chemical Analogues](#4-chemical-analogues)

5. [Formula Suggestions and Naming](#5-naming-and-formula-suggestions)

## 1. RAW file Conversion (Supported on Windows only):

This functionality will take in a collection of Thermo .RAW files containing 1 or more scans and combine them into a single .csv file, averaging the contained scans.  

To extract all the RAW files in a folder, `yourFolder` and write them to a new csv, `yourOutput.csv`, first, make a `DotRAWs`object:

 ~~~
 r = DotRAWs(inFolder = yourFolder, outPath = "yourOutput.csv")
 ~~~

Alternatively, to extract all RAW files from a list of paths, `yourList`:
~~~
r = DotRaws(inList = yourList, outPath = "yourOutput.csv")
~~~
Then, run the `getSpectra()` method:
~~~
 r.getSpectra()
~~~
 To extract and average the scans of a single RAW file, `inFile = "yourFile.RAW"`, use a similar process:
~~~
 r = DotRAWs(inFile = "yourFile.RAW")
 
 r.getSpectrum()
~~~
The `getSpectra()` and `getSpectrum()` methods will both return the path to the new csv file for later reference. If the `outputPath` was not specified, a default file name will be generated.



## 2. Peak Extraction
This functionality identifies all the relative peaks in each spectrum of a csv file and returns the m/z ratio and intensity value at each peak in each spectra in the form of another .csv file. 

 First, make a `Peaks` object with csv file `"yourSpectra.csv"` and optional output path `"yourOutput.csv"`:
 ~~~
  pi = Peaks(filePath = "yourSpectra.csv", outPath = "yourOutput.csv")
  ~~~
 Then, run the `isolatePeaks()` method on the object:
 ~~~
 pi.isolatePeaks()
~~~
 Like the extraction methods, the `isolatePeaks()` method will return the output path for later use. If the `outputPath` was not specified, a default file name will be generated.

## 3. Peak Processing

This feature comes with 3 main functionalities, both of which are accessible through the `PeakProcessing` class. Given multiple spectra, such as those produced in Peak Extraction, this object type has methods to convert intensities to relative intensities, to combine multiple spectra into a single one through averaging, and to compare control and treatment trials. 

### 3a. Relative Intensities

This functionality takes in a spectrum or group of spectra and converts into relative intensities using the intensity of the internal standard.

First, make a `PeakProcessing` object with a control and/or treatment csv file as well as an internal standard. The mass error is optional. Then, load the data and call the method with the optional output file. 
~~~
p = PeakProcessing(trtmntFilePath = "myTreatment.csv", ctrlFilePath = "myControl.csv", internalStandard = myInternalStandard, massError = myMassError)

p.loadData()

p.writeRelativeIntensities(outPath = "myOutput.csv")
~~~

### 3b. Averaging
This functionality will average n (see averaging methods for more detail) treatment spectra into a single spectrum. There are 2 different implementations for two different use cases. First, treatment spectra may need to be averaged in a repetitive grouping structure (see first image below). Second, treatment spectra may need to be averaged in a non-repetitive grouping structure (see second image below). Averaging can be done more than once to condense spectra and reduce computation time.

#### Case 1
If the averaging structure is repetitive, it can be visualized in a diagram similar to the following, which shows two steps of averaging:

![image](https://user-images.githubusercontent.com/51035699/162819693-23cf2f0e-17bb-4a8c-8346-b15f0a440992.png)

For spectra that need to be averaged in repetitive groupings of size n use the `consistentCombiner` method:
~~~
p = PeakProcessing(trtmntFilePath = "myTreatment.csv", ctrlFilePath = "myControl.csv", internalStandard = myInternalStandard, massError = myMassError)

p.loadData()

p.consistentCombiner(windowSize = n)
~~~
This method call will combine consecutive spectra that are within the window size. Therefore, if there are initially q spectra, there will be q/n after the averaging. 

#### Case 2
If the averaging structure is uneven, it may look more similar to the following diagram, which shows two steps of averaging:

![image](https://user-images.githubusercontent.com/51035699/162821672-c0fa91c6-ade5-4e5e-a16c-660c873cca35.png)

For unevenly grouped spectra,  use the `inconsistentCombiner` method, inputting a list of ranges in the format `[[0, i], [i, j], [j, k], ... [p, q]]` where q is the initial number of spectra, and letters i - p are indices of the spectra in myTreatment.csv (See exampleScript.py for an example):
~~~
p = PeakProcessing(trtmntFilePath = "myTreatment.csv", ctrlFilePath = "myControl.csv", internalStandard = myInternalStandard, massError = myMassError)

p.loadData()

p.inconsistentCombiner(windowSize = n)
~~~
### 3c. Comparison
This functionality compares a group of treatment spectra to control spectra and writes a csv containing the mz-values unique to the control along with their trends in intensity across a the different treatment spectra (see comparison methods for more detail). It is best used after combining spectra by time-point for the treatment data (see exampleScript.py). The output path is optional and a default will be assigned if left unspecified. 

~~~
p = PeakProcessing(trtmntFilePath = "myTreatment.csv", ctrlFilePath = "myControl.csv", internalStandard = myInternalStandard, massError = myMassError)

p.loadData()

#Combine Spectra Here

p.writeComparison()
~~~
## 4. Chemical Analogues

This functionality, given a list of m/z or mass values, will return the values that have a common difference indicative of an analogue relationship. Access this method through  a `PCA` object. The rounding is an optional argument that controls the level of rounding needed to consider a chemical analogue.
~~~
Analoguing = PCA(mzList = myMzList, rounding = myRounding, diffMin = myMinDiff, diffMax = myMaxDiff)

Analoguing.getAnalogues(outPath = "myOutput.csv")
~~~


## 5. Naming and Formula Suggestions
This functionality will suggest molecular formulas and verify that they exist using PubChem. Given a list of m/z values or masses, it will return a .csv file containing the m/z values, their formulas, mass errors, unsaturation degrees where applicable, and IUPAC names. To propose a formula, the software must have constraints on the number of elements in the compound, in the form of a dictionary. For instance: 

~~~
myElements = {'C': [1, 14], 'H': [0, 32], 'F': [1, 30], 'O': [0, 5], 'N':[0,5], 'S':[0,3], 'P':[0,3]}
~~~

If experiments require that the same elements be used repeatedly, it may be convenient to store the information in config.yml (see defaults and config) to avoid reconstructing dictionaries and having to pass them repeatedly. To minimize the number of API calls, the software also uses a local databse, that gets updated with information downloaded through each API call. The default location of the database can be found and modified in file config.yml (see defaults and config).
One way to instantiate a `MassList` object is by passing a list of mass or m/z values. If the list of floats represents m/z values, set mz to True:
~~~
m = MassList(elements = myElements, List = myList, massError = myMassError, dbPath = "myDB.csv", mz = True)
~~~
Another way to instantiate a `MassList` object is by passing a csv file that contains either a list of masses or a list of m/z values in its first column. Then, call the `loadData()` method to import the data from the csv. If the data is a list of m/z values, set `mz = True`, otherwise leave mz as false:
~~~
m = MassList(elements = myElements, filePath = "myFile.csv", massError = myMassError, dbPath = "myDB.csv")

m.loadData(mz = True)
~~~
From here, to get the information, run the `.search()` method, with optional output file path:
~~~
m.search(outPath = "myOutput.csv")
~~~
This call will return the path of the output csv.

# Key Definitions
These definitions are important in fully understanding the usage scenarios.
* [Mass Error](https://github.com/mb-2003/analysisProject/edit/main/README.md#mass-error)
* [Internal Standard and Relative Intensity](https://github.com/mb-2003/analysisProject/edit/main/README.md#internal-standard-and-relative-intensity)
* [Potential Chemical Analogue](https://github.com/mb-2003/analysisProject/edit/main/README.md#potential-chemical-analogue)
* [Trend Factor](https://github.com/mb-2003/analysisProject/edit/main/README.md#trend-factor)
* [Comparison Factor](https://github.com/mb-2003/analysisProject/edit/main/README.md#comparison-factor)


## Mass Error
A measure of error defined for two mz values:

![MME](https://latex.codecogs.com/png.image?\dpi{110}&space;\frac{|m/z_0&space;-&space;m/z_1|}&space;{m/z_0})

The mass error calculation is useful because it indicates how similar two peaks are. In the software, if two m/z or mass values have a mass error below the passed `massError` value, they are considered to be identical. 

## Internal Standard and Relative Intensity 

To convert intensity to relative intensity, the intensity of the internal standard must first be determined. Then, the relative intensity of each m/z value will be divided by the intensity of the internal standard:

![Relative Intensity](https://latex.codecogs.com/gif.latex?I_%7BRel%7D%20%3D%20%5Cfrac%7BI%7D%7BI_%7BIS%7D%7D)

Combination of spectra can be done without this step if no internal standard was used.

## Potential Chemical Analogue

In some samples, there will emerge a pattern some m/z values can be expressed as 

![Chemical Analogue](https://latex.codecogs.com/gif.latex?m/z_0%20&plus;%20nd)

where  ![m/z_0](https://latex.codecogs.com/gif.latex?m/z_0)  is a base m/z value, n is a non-negative integer, and d is a common difference.

The emergence of this pattern may suggest the existence of molecules with identical terminal groups but a variable chain length. 

## Trend Factor

When comparing the relative intensities across multiple timepoints and machine measurements, values within certain margins of each other are considered to be equal. For two intensity values, if they differ by a factor less than the one specified, they are reported as being approximately equal in the peak comparison step. 

## Comparison Factor

When comparing the relative intensities of between control and treatment samples, values in the treatment set must be greater than those of the control set by a factor greater than specified in order to be considered in the results. 




## Building Scripts

The indedendent functionalities of this software can be assembled in a data-processing pipeline, taking in sets of .RAW files and returning lists of compounds, potential analogues, and more. It is therefore important to understand how to assemble system, customized to specific use cases. See `exampleScripts` for sample scripts that integrate all the steps to process data.

## Defaults and Configuration

When a parameter is not specified in a method call, the software will default to the value stored in `config.yml`. The configuration file is helpful as it can reduce the number of paramaters that the user has to pass. For example, should the user not specify the mass error, the software will use the value for the mass error stored in `config.yml`. To modify the default values, edit them in configuration file, being sure to not change the file format, spacing, or indentation. 

## Other Feautures

### Local Database

When the software suggests formulas for elements, it has to make hundreds of API calls, consuming a significant amount of time. To mitigate this issue, the software also keeps a csv file `db.csv` that maintains a local record of the compounds already scraped from online databases. As the software identifies an increasing number of compounds, the database file will grow, minimizing the number of API calls further. In a similar way, the software also keeps track of molecular formulas for which there is no online record of discovery, enabling the software to skip over these formulas to reduce API calls. These unrecorded formulas are stored in `invalids.csv`.

