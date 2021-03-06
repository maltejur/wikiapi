﻿[![npm version](https://badge.fury.io/js/wikiapi.svg)](https://www.npmjs.com/package/wikiapi)
[![npm downloads](https://img.shields.io/npm/dm/wikiapi.svg)](https://www.npmjs.com/package/wikiapi)
[![Build Status](https://travis-ci.org/kanasimi/wikiapi.svg?branch=master)](https://travis-ci.org/kanasimi/wikiapi)
[![codecov](https://codecov.io/gh/kanasimi/wikiapi/branch/master/graph/badge.svg)](https://codecov.io/gh/kanasimi/wikiapi)

[![Known Vulnerabilities](https://snyk.io/test/github/kanasimi/wikiapi/badge.svg?targetFile=package.json)](https://snyk.io/test/github/kanasimi/wikiapi?targetFile=package.json)
[![codebeat badge](https://codebeat.co/badges/47d3b442-fd49-4142-a69b-05171bf8fe36)](https://codebeat.co/projects/github-com-kanasimi-wikiapi-master)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/02aa4b9cc9df4fa9b10389abbb139ebf)](https://app.codacy.com/app/kanasimi/wikiapi?utm_source=github.com&utm_medium=referral&utm_content=kanasimi/wikiapi&utm_campaign=Badge_Grade_Dashboard)
[![DeepScan grade](https://deepscan.io/api/teams/4788/projects/6757/branches/58325/badge/grade.svg)](https://deepscan.io/dashboard#view=project&tid=4788&pid=6757&bid=58325)

# JavaScript MediaWiki API
A simple way to access MediaWiki API via JavaScript with simple wikitext parser.
This is basically a modern syntax version of [CeJS MediaWiki module](https://github.com/kanasimi/CeJS/blob/master/application/net/wiki). For example, using [async functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function).

## Features
*   Read / edit pages.
*   Get list of categorymembers, pages transclude specified template, and more...
*   Auto-limited editing rate.
*   Parse wikitext / pages. You may modify parts of the wikitext, then regenerate the page just using .toString().

## Installation
Install [node.js](https://nodejs.org/) first.

```bash
npm install wikiapi
```

## Usage
Here lists some examples of this module.

### As node.js module
```javascript
// load module
const Wikiapi = require('wikiapi');

// login
(async () => {
	const wiki = new Wikiapi(user, password, 'en');
	//wiki = new Wikiapi(user_name, password, API_URL);
	// Calling in another domain
	//wiki = new Wikiapi({user_name:'', password:'', origin:'*'});
})();

// load page
(async () => {
	// on Wikipedia...
	const wiki = new Wikiapi('en');
	// ...or other MediaWiki websites
	//const wiki = new Wikiapi('https://awoiaf.westeros.org/api.php');
	let page_data = await wiki.page('Universe');
	console.log(page_data.wikitext);
})();

// Get multi revisions
(async () => {
	const wiki = new Wikiapi;
	let page_data = await wiki.page('Universe', {
		// Get multi revisions
		revisions: 2
	});
	console.log(page_data.wikitext);
})();

// edit page
(async () => {
	const enwiki = new Wikiapi;
	await enwiki.login('bot name', 'password', 'en');
	let page_data = await enwiki.page('Wikipedia:Sandbox');
	await enwiki.edit(function(page_data) {
		return page_data.wikitext
			+ '\nTest edit using {{GitHub|kanasimi/wikiapi}}.';
	}, {bot: 1});
	console.log('Done.');
})();

// parse wiki page (The parser is more powerful than the example. Try yourself!)
(async () => {
	// Usage with other language
	const zhwiki = new Wikiapi('zh');
	await zhwiki.login('user', 'password');
	let page_data = await zhwiki.page('Universe');
	// Other types:
	// @see wiki_toString @ https://github.com/kanasimi/CeJS/blob/master/application/net/wiki/parser.js
	page_data.parse().each('template',
		token => console.log(token.name));
})();

// read wikidata
(async () => {
	const wiki = new Wikiapi;
	let page_data = await wiki.data('Q1');
	// Work with other language
	console.assert(CeL.wiki.data.value_of(page_data.labels.zh) === '宇宙');
})();

// read wikidata
(async () => {
	const wiki = new Wikiapi;
	let data = await wiki.data('Universe', 'P1419');
	console.assert(data.includes('shape of the universe'));
})();

// get list of [[w:en:Category:Chemical_elements]]
(async () => {
	const wiki = new Wikiapi;
	let list = await wiki.categorymembers('Chemical elements');
	console.log(list);
	// Working on multiple pages
	await wiki.for_each_page(
		/* {Array} title liat / page data list */ list,
		page_data => { /* ... */ });
})();

// get pages transcluding {{w:en:Periodic table}}
(async () => {
	const wiki = new Wikiapi;
	let list = await wiki.embeddedin('Template:Periodic table');
	console.log(list);
})();

// upload media
(async () => {
	const wiki = new Wikiapi;
	await wiki.login('user', 'password', 'test');
	// Upload a local file directly:
	let result = await wiki.upload_file({ file_path: '/local/file/path', comment: '', text: '' });
	// Upload file from URL:
	result = await wiki.upload_file({ media_url: 'https://media.url/name.jpg', comment: '', text: '' });
})();

// update wikidata
(async () => {
	// Just for test
	delete CeL.wiki.query.default_maxlag;
	const wiki = new Wikiapi;
	await wiki.login('user', 'password', 'test');

	// Get https://test.wikidata.org/wiki/Q7
	let entity = await wiki.data('Q7');
	// search [ language, label ]
	//entity = await wiki.data(['en', 'Earth']);

	// Update claim
	await entity.modify({ claims: [{ P17: 'Q213280' }] });
	// Update claim: set country (P17) to 'Test Country 1' (Q213280) ([language, label] as entity)
	await entity.modify({ claims: [ { language: 'en', country: [, 'Test Country 1'] } ] });
	// Remove country (P17) : 'Test Country 1' (Q213280)
	await entity.modify({ claims: [ { language: 'en', country: [, 'Test Country 1'], remove: true } ] });

	// Update label
	await entity.modify({ labels: [ { language: 'zh-tw', value: '地球' } ] });
})();
```

More examples: Please see [test.js](https://github.com/kanasimi/wikiapi/blob/master/_test%20suite/test.js).

## OS support
| Platform    | support |
| ----------- | ------- |
| Windows     | ✔️       |
| macOS       | ✔️       |
| UNIX, Linux | ✔️       |

## See also
For old style JavaScript, or general environment usage, please see [wikibot](https://github.com/kanasimi/wikibot).

## Contact
Contact us at [GitHub](https://github.com/kanasimi/wikiapi/issues).

[![logo](https://raw.githubusercontent.com/kanasimi/CeJS/master/_test%20suite/misc/logo.jpg)](http://lyrics.meicho.com.tw/)
