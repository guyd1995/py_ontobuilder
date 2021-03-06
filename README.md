# Ontobuilder

This is a Python port to [Ontobuilder](https://github.com/shraga89/ontobuilderDev).

This can actually be modified to work with basically any Java package in JAR format by replacing the contents of `src/ontobuilder/jars` and adapting `src/ontobuilder/__init__.py` and `setup.py` accordingly. 


## Installation
### Conda
1. (**Optional**) Create new environment (might be useful if package conflicts with other packages arise in step 2):
```
conda create -n ontobuilder python
conda activate ontobuilder
```

2. Install dependecies
```
conda install -c conda-forge -y openjdk>=11.0.9 maven jpype1
```
In case of problems try explicitly stating the following versions:

jpype1=1.3.0  
maven=3.6.3  
openjdk=11.0.9.1  

3. Then simply run:
```
pip install -e git+https://github.com/dar-tau/py_ontobuilder.git@main#egg=ontobuilder
```

## Package Structure
The main three submodules are: **ontobuilder.core**, **ontobuilder.matching**, **ontobuilder.io**. However, in Python, unlike Java, we prefer less hierarchical submodule paths. 
Consequently, we decided to flatten out the module structure by extracting important submodules to the second layer of the hierarchy (namely, directly under ontobuilder):
* **ontobuilder.core**
* **ontobuilder.matching**
* **ontobuilder.io**
* **ontobuilder.ontology** (ontobuilder.core.ontology)
* **ontobuilder.imports** (ontobuilder.io.imports)
* **ontobuilder.exports** (ontobuilder.io.exports)
* **ontobuilder.algorithms** (ontobuilder.matching.algorithms)
* **ontobuilder.wrapper** (ontobuilder.matching.wrapper)

## Examples
**Matching ontologies**:
```python
from ontobuilder.ontology import Ontology, Term
from ontobuilder.wrapper import OntoBuilderWrapper

obw = OntoBuilderWrapper(True)
o1 = Ontology("Test1", "Title Test 1")
o2 = Ontology("Test2", "Title Test 2")
o1.setLight(True)
o2.setLight(True)

o1.addTerm(Term("airplane", "Skipper"))
o2.addTerm(Term("aircraft", "B-52"))

res1 = obw.matchOntologies(o1, o2, "Term Match")
res2 = obw.matchOntologies(o1, o2, "WordNet Match")
```

**Importing ontologies from files**: Sadly we must use Java's `File` object rather than the pythonic `open`. To run the following example you will need to download Apertum.xsd from [here](https://raw.githubusercontent.com/shraga89/ontobuilderDev/maven/ontobuilder.io/src/test/resources/Apertum.xsd).

```python
from ontobuilder.imports import XSDImporterUsingXerces
import jpype.imports # this will allow for calling java imports directly from python
from java.io import File # this is a Java import 

importer = XSDImporterUsingXerces()
f = File("./Apertum.xsd")
o = importer.importFile(f)

assert o.getAllTermsCount() == 144	

```

## Debugging
* **When importing a nonexistent object from module**, e.g.: 
```python
from ontobuilder.wrapper import blabla
```
the exception that will be raised is 
```
ModuleNotFoundError: No module named 'ac'
```
which is irrelevant. Until I fix that, please be aware of this.

* **You might need to use [JPype](https://github.com/jpype-project/jpype)**. I tried to keep this to a minimum, but in cases where you do need here are some main tips.  

You can import from Java package directly from Python after running:
```python
import jpype.imports
```
Now packages in Java are available for you (specifically, they are required to be inside of `java`, `com`, `org`, `edu`, `mil` and maybe a few more), eg.:
```python
from java.io import File
import org.jdom
```

In case you need Java types, you can import them from `jpype.types`:
```python
from jpype.types import JString
```

* **If you encounter JVM problems**: This is not supposed to happen. But if you see

```
OSError: JVM is already started
```

This means that you tried to start another JVM through jpype while ontobuilder's JVM is already running (or vice versa). This might happen if another package using jpype is imported. Currently, there's no easy solution, but I might solve this later if required.


## Song
"  
Ontobuilder.  
Qu'est-ce que c'est?  
Pa-Pa-Pa Pa-Pa-Python - far better!  
Run, run, run, run... plug and play!  
"
