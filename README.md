# api documentation for  [ps-node (v0.1.5)](https://github.com/neekey/ps#readme)  [![npm package](https://img.shields.io/npm/v/npmdoc-ps-node.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-ps-node) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-ps-node.svg)](https://travis-ci.org/npmdoc/node-npmdoc-ps-node)
#### A process lookup utility

[![NPM](https://nodei.co/npm/ps-node.png?downloads=true)](https://www.npmjs.com/package/ps-node)

[![apidoc](https://npmdoc.github.io/node-npmdoc-ps-node/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-ps-node_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-ps-node/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-ps-node/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-ps-node/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "bugs": {
        "url": "https://github.com/neekey/ps/issues"
    },
    "dependencies": {
        "table-parser": "^0.1.3"
    },
    "description": "A process lookup utility",
    "devDependencies": {
        "mocha": "^2.4.5"
    },
    "directories": {},
    "dist": {
        "shasum": "b51e5dd5650fe12ab4785d76ac4770dfbc56f986",
        "tarball": "https://registry.npmjs.org/ps-node/-/ps-node-0.1.5.tgz"
    },
    "gitHead": "720798dc1d59be6dd2558da5b41229ada5de8c42",
    "homepage": "https://github.com/neekey/ps#readme",
    "keywords": [
        "ps",
        "process",
        "lookup",
        "pid"
    ],
    "license": "MIT",
    "main": "index.js",
    "maintainers": [
        {
            "name": "neekey",
            "email": "ni184775761@gmail.com"
        }
    ],
    "name": "ps-node",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git://github.com/neekey/ps.git"
    },
    "scripts": {
        "test": "node ./node_modules/mocha/bin/mocha -t 0 -R spec test/test.js",
        "testWatch": "node ./node_modules/mocha/bin/mocha -t 0 -R spec --watch test/test.js"
    },
    "version": "0.1.5"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module ps-node](#apidoc.module.ps-node)
1.  [function <span class="apidocSignatureSpan">ps-node.</span>kill ( pid, signal, next )](#apidoc.element.ps-node.kill)
1.  [function <span class="apidocSignatureSpan">ps-node.</span>lookup (query, callback)](#apidoc.element.ps-node.lookup)



# <a name="apidoc.module.ps-node"></a>[module ps-node](#apidoc.module.ps-node)

#### <a name="apidoc.element.ps-node.kill"></a>[function <span class="apidocSignatureSpan">ps-node.</span>kill ( pid, signal, next )](#apidoc.element.ps-node.kill)
- description and source-code
```javascript
kill = function ( pid, signal, next ){
  //opts are optional
  if(arguments.length == 2 && typeof signal == 'function'){
    next = signal;
    signal = undefined;
  }

  if (typeof signal === 'object') {
    signal = signal.signal;
  }

  try {
    process.kill(pid, signal);
  } catch(e) {
    return next && next(e);
  }

  var checkConfident = 0;

  function checkKilled(finishCallback) {
    console.log('check kill success', pid, 'confident', checkConfident);
    exports.lookup({ pid: pid }, function(err, list) {
      console.log('check kill success result:', (list || []).length);
      if (err) {
          finishCallback && finishCallback(err);
        } else if(list.length > 0) {
          checkKilled(finishCallback);
        } else {
          checkConfident++;
          if (checkConfident === 5) {
            finishCallback && finishCallback();
          } else {
            checkKilled(finishCallback);
          }
        }
    });
  }

  next && checkKilled(next);
}
```
- example usage
```shell
...

Also, you can use 'kill' to kill process by 'pid':

'''javascript
var ps = require('ps-node');

// A simple pid lookup
ps.kill( '12345', function( err ) {
    if (err) {
        throw new Error( err );
    }
    else {
        console.log( 'Process %s has been killed!', pid );
    }
});
...
```

#### <a name="apidoc.element.ps-node.lookup"></a>[function <span class="apidocSignatureSpan">ps-node.</span>lookup (query, callback)](#apidoc.element.ps-node.lookup)
- description and source-code
```javascript
lookup = function (query, callback) {

<span class="apidocCodeCommentSpan">  /**
   * add 'l' as default ps arguments, since the default ps output in linux like "ubuntu", wont include command arguments
   */
</span>  var exeArgs = query.psargs || ['l'];
  var filter = {};
  var idList;

  // Lookup by PID
  if (query.pid) {

    if (Array.isArray(query.pid)) {
      idList = query.pid;
    }
    else {
      idList = [query.pid];
    }

    // Cast all PIDs as Strings
    idList = idList.map(function (v) {
      return String(v);
    });

  }


  if (query.command) {
    filter['command'] = new RegExp(query.command, 'i');
  }

  if (query.arguments) {
    filter['arguments'] = new RegExp(query.arguments, 'i');
  }

  if (query.ppid) {
    filter['ppid'] = new RegExp(query.ppid);
  }

  return Exec(exeArgs, function (err, output) {
    if (err) {
      return callback(err);
    }
    else {
      var processList = parseGrid(output);
      var resultList = [];

      processList.forEach(function (p) {

        var flt;
        var type;
        var result = true;

        if (idList && idList.indexOf(String(p.pid)) < 0) {
          return;
        }

        for (type in filter) {
          flt = filter[type];
          result = flt.test(p[type]) ? result : false;
        }

        if (result) {
          resultList.push(p);
        }
      });

      callback(null, resultList);
    }
  });
}
```
- example usage
```shell
...

Lookup process with specified 'pid':

'''javascript
var ps = require('ps-node');

// A simple pid lookup
ps.lookup({ pid: 12345 }, function(err, resultList ) {
if (err) {
    throw new Error( err );
}

var process = resultList[ 0 ];

if( process ){
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
