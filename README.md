# Machine Learning for Open source code scan
## Anomaly Dectection of malicious packages.
 The Backstabber's knife dataset provides the collection of malicious packages in different programming languages. 
 The majority of packages in this dataset come from the npm repository. The motivation of this work is based on the fact that among the 1486 malicious JavaScript packages only 2  of them are detected as malicious ,in a static fashion, (Trojan attacks ) by the Window McAfee Endpoint Security (flatmap-stream and klow). Therefore new solutions are needed in order to produce static analysis of open source packages. 

We consider a collection of packages in order to create a labelled dataset: 1 if the packages is malicious, 0 if is benign. 
 The aim of the project is to cover the all surface of an open source package to inspect malicious injections and to investigate machine learning models to seek new solutions. 
 With this in mind we divided into three branches the files that contain the code/text of a package:
 
 - Markdown sub-files: despite they are mostly used as a getting-started framework for the users, or to track the changes and updates of the packages, the majority of malicious packages do not have a README.md file. The presence/absence of at least one markdown file in the package may be helpful in discriminating between benign and malicious packages.
 - JSON file: all npm packages contain a file, usually in the project root, called package.json. This file holds various metadata relevant to the project. It is used to give information to npm that allows it to identify the project, handle project's dependencies and specifying name and contacts of the maintainers. It can be used also to express some preinstall and postinstall rules. 
 - JavaScript source code sub-files: npm packages are mainly construct by file.js. We unified the code coming from different sub-files creating a unique main-file.js.

### Javascript 
  We look at a JavaScript lexer (slimit) in order to create and inspect the code from an higher level point of view. At this regards we inspect some high-level token type after having aggregated together all file.js in the package:
   - identifiers:  a sequence of characters in the code that identifies a variable, function, or property.
   - numbe of regex (regular expression): sequence of characters used for defining a search pattern  https://blog.malwarebytes.com/security-world/technology/2018/08/explained-regular-expression-regex/
   - strings. 
   - number of square brackets. 
   - number of ;
   In addition we also take in account the number of sub-file.js in the package. 
   
### JSON 
 We investigate the json.loads method to parse JSON strings into python dictionaries. We focus on the package.json : 
 - script key: which can include subkeys such as preinstall and postinstall.   
 - number of words in the package.json.  

From this preliminary investigation we discover that some packages from npm in the Backstabber's collection does not contain any file.js . These packages will not be consider in the analysis. Furthermore some packages have an incorrect JSON format, which are difficult to scan and analyzing (discord-fix, misc, momnet,pm-controls, radicjs, sailclothjs). In addition the Backstabber's collection provides an indication of the campaign of attack in which the package belong. Since packages within the same campaign may have the same malicious code injected we just consider one package for each campaign. We consider only last available version of the infected package.   

### Markdown: 
- boolean: (presence/absence) of at least one markdown file.

### Construction of the collection of benign packages:
- Our assumption is that, with this methodology, we are able to select from the npm repository benign packages, by exploting a trade off between different domain of the package and user-preference. We use pybraries, which is a Python wrapper for the libraries.io API. We search for projects with a certain  keyword, JavaScript and NPM  as language and platform, sort by the number of stars and rank. We select 150 keywords for the domain of the packages and for each of them we select the ones with the highest rank and stars.  After that we drop packages with a number of stars lower than a certain threshold. To decide the threshold we look at the twentieth percentile of the distribution of the number of stars. We take last available version of each package as for the malicious ones.

The list of 150 keywords: 'math','write','path','google','virtual','supervised','screen','time','box','inject','domain', 'folder','service','mobile','convert', 'method','source','cloud','signature','daily','filter','chat','iot','standard','style','query','testing','documentation','robotics', 'planning','supply chain', 'food','search','çss','back-end','frameworks','physics','cli', 'media', 'scraping','id','front-end','battery','aws', 'azure',
 'tree','server','health','net','marketing','nlp','compiler','load', 'UI','database','statistic','science','web','nutrition','diet','utilities', 'react','typings',
 'response','monitor','notification', 'town','transpose', 'music', 'dictionary', 'scale', 'feature', 'lexer', 'optimize', 'vector', 'matrix', 'icon', 'analytics','universal','cognito','template','tool','microsoft','interface', 'prompt','hyper', 'api', 'cache', 'bash', 'image', 'plugin', 'detection', 'system', 'blockchain','open','plot','dash','finance', 'next','code', 'npm','proxy','array','class','instance','object', 'binary','executable','map','shell','input','game','node','core','fastify','linux','sensor','call','serial','information', 'algorithm','engine','byte','python', 'sync','javascript', 'js','url','check','fuzzing', 'app','build', 'sql','parameters','visualization', 'pool','access', 'direct', 'git','language','global','config',
 'function','print','update', 'render', 'unified','clause','cookies','host'. 
 
 ### Imbalanced dataset from the collection of packages. 
 The labelled dataset contains 1080 packages, 964 benign and 116 malicious. Malicious packages represents only the 10% of the entire dataset, going in the direction of simulating a real situation where hopefully malicious packages are just a minority among all open source packages. 
 
 #### Approach 
 Our methodology aim at extracting, in an automated fashion, features from the inspected surface of the package and deploy a DecisionTreeclassifier model. The approach requires the following steps:
 
 1) Definition of a set of generalization languages, which can be applied in different contexts (strings, identiifers, script key in package.json). A generalization language is just a hierarchy tree that maps code to a new representation (a new alphabet). Each string, identifier, or object in the script key in package.json is translated to a generalization language.  
 2) Definition of  sets of stopwords, which can be removed from different contexts (strings, script key in package.json) before applying the generalization language. This set stopwords mostly rely on english words, with the exception of: linux command that can be used to injects malicious payloads (bash,curl,tap), install rules (preinstall,postinstall) and some potentially dangerous libraries (network). Among the sets of stopwords there is also an empty set of words.  
 3) Application of the Shannon Entropy to the generalization language choosen for a specific context. It is used to measure of the amount of information in a given context. A  measure of the uncertainty can help in discriminating patterns that depart heavily from the core of the distribution. 
 4) Extract features from shannon entropy computation for each packages: mean,standard deviation, maximum and third quartile in each context (strings, identifiers, script key in package.json)
 5) Extract features that do not depend on shannon entropy: number of js.file, number of words in package.json, number of regular expression semicolumns and square brackets in the main-file.js, presence/absence of at least one markdown file. 

#### Methodology for parameter optimization 
The parameter optimization aims at replying at the following questions:
- Which generalization language fit better a given context?
- Make sense to remove stopwords in a given context? Or the best set of stopwords is the empty set?
- Which is the most appropriate base in the computation of the shannon entropy? 
- Which is the best set of hyperparameters of the DecisionTreeclassifier to detect malicious packages? 

In order to solve these issues we propose a Bayesian Optimization, which is an informed search such that the next set of parameters to be evaluated depends on the previous search. This methodology tries to explore regions of the parameters space where we are getting good results rather than explore all possible combinations. The optimization is applied not only to the hyperparameters of the classifiers but also to parameters (generalization languages, stopwords, base of the logarithm) that influences directly or indirectly the features from the shannon entropy computation. 



