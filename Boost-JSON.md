<div id="page">

<div id="main" class="aui-page-panel">

<div id="main-header">

<div id="breadcrumb-section">

1.  <span>[Data Science](index.html)</span>

</div>

# <span id="title-text">Data Science : JSON Parsing with Boost Property Tree</span>

</div>

<div id="content" class="view">

<div class="page-metadata">Created by <span class="author">Akhtar, Zubair</span>, last modified on Mar 06, 2019</div>

<div id="main-content" class="wiki-content group">

<span style="color: rgb(0,0,0);">Boost Property Tree library provides a data structure that stores an arbitrarily deeply nested tree of values, indexed at each level by some key. Each node of the tree stores its own value, plus an ordered list of its sub-nodes and their keys. The tree allows easy access to any of its nodes by means of a path, which is a concatenation of multiple keys. T</span>he library also provides parsers and generators for a number of data formats that can be represented by such a tree, including XML, INI, and JSON. I will cover JSON parsing in this post.

### Anatomy & Accessors

Conceptually, the property tree resembles a standard container with value type of std::pair<string, ptree>. It provides its own, tree-specific interface, and each node is also an STL-compatible sequence for its child nodes. An important thing to note here is that the property sequence is not ordered by the key. It preserves the order of insertion. The property tree has the usual member functions, such as [insert](https://www.boost.org/doc/libs/1_65_1/doc/html/boost/property_tree/basic_ptree.html#idp699497824-bb), [push_back](https://www.boost.org/doc/libs/1_65_1/doc/html/boost/property_tree/basic_ptree.html#idp699513872-bb), [find](https://www.boost.org/doc/libs/1_65_1/doc/html/boost/property_tree/basic_ptree.html#idp699533920-bb), [erase](https://www.boost.org/doc/libs/1_65_1/doc/html/boost/property_tree/basic_ptree.html#idp699505968-bb), etc which can be used to populate and access the tree.  

<div class="code panel pdl" style="border-width: 1px;">

<div class="codeHeader panelHeader pdl" style="border-bottom-width: 1px;">**Example 1**</div>

<div class="codeContent panelContent pdl">

<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: cpp; gutter: true; theme: Confluence" data-theme="Confluence">#include <boost/property_tree/ptree.hpp>

namespace pt = boost::property_tree;

// Insert value
pt::ptree tree;
tree.push_back(pt::ptree::value_type("pi", pt::ptree("3.14159")));

// Retrieve value
ptree::const_iterator it = pt.find("pi");
double pi = boost::lexical_cast<double>(it->second.data());</pre>

</div>

</div>

For most practical purposes, using STL container methods for property trees is cumbersome. For ease of use, the property tree library provides two families of member functions, get and put, which allow intuitive access to data stored in the tree (direct children or not).

<div class="code panel pdl" style="border-width: 1px;">

<div class="codeHeader panelHeader pdl" style="border-bottom-width: 1px;">**Example 2**</div>

<div class="codeContent panelContent pdl">

<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: cpp; gutter: true; theme: Confluence" data-theme="Confluence">#include <boost/property_tree/ptree.hpp>

namespace pt = boost::property_tree;

pt::ptree tree;
tree.put("pi", 3.14159);                // put double
double pi = tree.get<double>("pi");     // get double

tree.put("pi", 3.14160);				// Overrides existing value
tree.add("pi", 3.15160);				// Creates a new node with duplicate key</pre>

</div>

</div>

The key takeaways from this example are:

*   Calling [put](https://www.boost.org/doc/libs/1_65_1/doc/html/boost/property_tree/basic_ptree.html#idp699660208-bb) will insert a new value at specified path, so that a call to [get](https://www.boost.org/doc/libs/1_65_1/doc/html/boost/property_tree/basic_ptree.html#idp699625152-bb) specifying the same path will retrieve it. The get call locates the proper node in the tree and tries to translate its data string to a float value. If path does not exist, locating data will fail and it will throw ptree_bad_exception. If value could not be translated correctly, it will throw [ptree_bad_data](https://www.boost.org/doc/libs/1_65_1/doc/html/boost/property_tree/ptree_bad_data.html). Both of them derive from [ptree_error](https://www.boost.org/doc/libs/1_65_1/doc/html/boost/property_tree/ptree_error.html) to make common handling possible.
*   P[ut](https://www.boost.org/doc/libs/1_65_1/doc/html/boost/property_tree/basic_ptree.html#idp699660208-bb) will insert any missing path elements during path traversal. For example, calling [put](https://www.boost.org/doc/libs/1_65_1/doc/html/boost/property_tree/basic_ptree.html#idp699660208-bb)("key1.key2.key3",3.14f) on an empty tree will insert three new children: key1, key1.key2 and key1.key2.key3\. The last one will receive a string "3.14" as data, while the two former ones will have empty data strings. [put](https://www.boost.org/doc/libs/1_65_1/doc/html/boost/property_tree/basic_ptree.html#idp699660208-bb) always inserts new keys at the back of the existing sequences.

### JSON Parsing

The property tree library provides read_json and write_json methods to parse and write JSON structures. Both methods have different variants to allow the flexibility of manipulating JSON structures with istream/ostream or simply with a filename.

<div class="code panel pdl" style="border-width: 1px;">

<div class="codeHeader panelHeader pdl" style="border-bottom-width: 1px;">**Example 3**</div>

<div class="codeContent panelContent pdl">

<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: cpp; gutter: true; theme: Confluence" data-theme="Confluence">#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/json_parser.hpp>
#include <sstream>

std::istringstream is{
R"(
{
	"a": 1,
	"b": [{
		"b_a": 2,
		"b_b": {
			"b_b_a": "val1",
			"b_b_b": "val2"
		},
		"b_c": 0,
		"b_d": [{
			"b_d_a": 3,
			"b_d_b": {
				"b_d_b_a": 4
			},
			"b_d_c": "test",
			"b_d_d": {
				"b_d_d_a": 5
			}
		}],
		"b_e": null,
		"b_f": [{
			"b_f_a": 6
		}],
		"b_g": 7
	}],
	"c": 8
}
)" };

// To read this JSON structure into a property tree: 
boost::property_tree::read_json(is, pt);

// Can also be done as:
boost::property_tree::read_json("xyz.json", pt);</pre>

</div>

</div>

As shown in Example 2, reading the variables at root level is trivial. It can be done as simply as:

<div class="code panel pdl" style="border-width: 1px;">

<div class="codeHeader panelHeader pdl" style="border-bottom-width: 1px;">**Example 3a**</div>

<div class="codeContent panelContent pdl">

<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: cpp; gutter: true; theme: Confluence" data-theme="Confluence">int v1 = pt.get<int>("a");
int v2 = pt.get<int>("c");</pre>

</div>

</div>

Easy right? Now lets try to get the value of "b_b_a". Accessing random nodes inside b is slightly tricky as b is a JSON array. <span style="color: rgb(0,0,0);">Therefore, one has to iterate over the list:</span>

<div class="code panel pdl" style="border-width: 1px;">

<div class="codeHeader panelHeader pdl" style="border-bottom-width: 1px;">**Example 3b**</div>

<div class="codeContent panelContent pdl">

<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: cpp; gutter: true; theme: Confluence" data-theme="Confluence">const boost::property_tree::ptree& b = pt.get_child("b");
for (const auto& kv : b) 
{ 
	std::cout << "b_b_a = " << kv.second.get<std::string>("b_b.b_b_a") << "\n"; 
}</pre>

</div>

</div>

Building on this example, the value of "b_d_d_a" can be obtained as:

<div class="code panel pdl" style="border-width: 1px;">

<div class="codeHeader panelHeader pdl" style="border-bottom-width: 1px;">**Example 3c**</div>

<div class="codeContent panelContent pdl">

<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: cpp; gutter: true; theme: Confluence" data-theme="Confluence">#include <boost/foreach.hpp>

const boost::property_tree::ptree& b = pt.get_child("b");
for (const auto& kv : b) 
{ 
	BOOST_FOREACH(const boost::property_tree::ptree::value_type &v, kv.second.get_child("b_d")) 
	{ 
		std::cout << "b_d_d_a = " << v.second.get<std::string>("b_d_d.b_d_d_a") << "\n"; 
	}
} </pre>

</div>

</div>

Well that was not so difficult because the array in "b" had one element and we only looked inside it to find "b_d" and "b_b". If there are more elements in the array, the get_child calls in these examples will throw exceptions. Lets modify the JSON structure from Example 3 to include more elements in the array.

<div class="code panel pdl" style="border-width: 1px;">

<div class="codeHeader panelHeader pdl" style="border-bottom-width: 1px;">**Example 4**</div>

<div class="codeContent panelContent pdl">

<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: cpp; gutter: true; theme: Confluence" data-theme="Confluence">#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/json_parser.hpp>
#include <sstream>

std::istringstream is{
R"(
{
	"a": 1,
	"b": [{
		"b_a": 2,
		"b_b": {
			"b_b_a": "val1",
			"b_b_b": "val2"
		},
		"b_c": 0,
		"b_d": [{
			"b_d_a": 3,
			"b_d_b": {
				"b_d_b_a": 4
			},
			"b_d_c": "test",
			"b_d_d": {
				"b_d_d_a": 5
			}
		}],
		"b_e": null,
		"b_f": [{
			"b_f_a": 6
		}],
		"b_g": 7
	},
	{ 
		"b_h": 9 
	}, 
	{ 	"b_i": 10 
	}],
	"c": 8
}
)" };</pre>

</div>

</div>

Notice the addition of "b_h" and "b_g". Since we cant make an assumption on where in the array our desired element is, we need to check each iteration. Fortunately, boost provides some nifty constructs to retrieve the value "b_b.b_b_a" from the JSON structure in Example 4.

#### Using Property Tree Methods:

This version uses boost::optional class to handle extraction failure. On successful extraction, it will return boost::optional initialized with extracted value. Otherwise, it will return uninitialized boost::optional. Since the returned optional value is a ptree, to retrieve a value from this tree use `[get_value](https://www.boost.org/doc/libs/1_65_1/doc/html/boost/property_tree/basic_ptree.html#idp699583712-bb)` or `[get_value_optional](https://www.boost.org/doc/libs/1_65_1/doc/html/boost/property_tree/basic_ptree.html#idp699609520-bb)`. They have identical semantics to `[get](https://www.boost.org/doc/libs/1_65_1/doc/html/boost/property_tree/basic_ptree.html#idp699625152-bb)` functions, except they don't take the `[path](https://www.boost.org/doc/libs/1_65_1/doc/html/boost/property_tree/string_path.html "Class template string_path")` parameter.

<div class="code panel pdl" style="border-width: 1px;">

<div class="codeHeader panelHeader pdl" style="border-bottom-width: 1px;">**Example 4a**</div>

<div class="codeContent panelContent pdl">

<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: cpp; gutter: true; theme: Confluence" data-theme="Confluence">#include <boost/optional.hpp>

const boost::property_tree::ptree& b = pt.get_child("b");
for (const auto& kv : b) 
{ 
	boost::optional< const boost::property_tree::ptree& > element = kv.second.get_child_optional("b_b.b_b_a"); 
	if (element) 
	{ 
		std::cout << "b_b_a = " << element->get_value<string>() << "\n";

		// Can also be done as
		std::cout << "b_b_a = " << element->data() << "\n"; 
	} 
}</pre>

</div>

</div>

#### Using STL style container methods:

<div class="code panel pdl" style="border-width: 1px;">

<div class="codeHeader panelHeader pdl" style="border-bottom-width: 1px;">**Example 4b**</div>

<div class="codeContent panelContent pdl">

<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: cpp; gutter: true; theme: Confluence" data-theme="Confluence">const boost::property_tree::ptree& b = pt.get_child("b"); 
for (const auto& kv : b) 
{ 
	auto it = kv.second.find("b_b"); 
	if (it != kv.second.not_found()) 
	{ 
		std::cout << "b_b_a = " << it->second.get<std::string>("b_b_a") << "\n"; 
	} 
} </pre>

</div>

</div>

### Printing Property Tree

<span style="color: rgb(36,39,41);">I've included code to print the whole tree recursively so the readers can understand and visualize how the JSON structure gets translated to the ptree. Arrays elements are stored as key/value pairs where the key is an empty string.</span>

<div class="code panel pdl" style="border-width: 1px;">

<div class="codeHeader panelHeader pdl" style="border-bottom-width: 1px;">**Example 5**</div>

<div class="codeContent panelContent pdl">

<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: cpp; gutter: true; theme: Confluence" data-theme="Confluence">#include <iostream> 
#include <sstream> 
#include <boost/property_tree/ptree.hpp> 
#include <boost/property_tree/json_parser.hpp> 

namespace pt = boost::property_tree; 

void print_ptree( const pt::ptree& tree, int level = 0 ) 
{ 
	if( tree.empty() ) 
	{ 
		cout << '"' << tree.get_value<std::string>() << "\"\n"; 
	} 
	else 
	{ 
		cout << '\n'; 
		for( const auto& kv : tree ) 
		{ 
			std::cout << std::string( level, '\t' ) << '"' << kv.first << "\": "; print_ptree( kv.second, level + 1 ); 
		} 
	} 
}</pre>

</div>

</div>

This will print our original JSON structure from Example 3 as:

<div class="preformatted panel" style="border-width: 1px;">

<div class="preformattedContent panelContent">

<pre>"a": "1"
"b": 
	"": 
		"b_a": "2"
		"b_b": 
			"b_b_a": "test"
		"b_c": "0"
		"b_d": 
			"": 
				"b_d_a": "3"
				"b_d_b": 
					"b_d_c": "4"
				"b_d_c": "test"
				"b_d_d": 
					"b_d_d": "5"
		"b_e": "null"
		"b_f": 
			"": 
				"b_f_a": "6"
		"b_g": "7"
"c": "8"</pre>

</div>

</div>

### Boost for Magiclamp

Boost property tree requires building with boost-serialization library which is currently not being used in Magiclamp. However, it is something we plan to add in future. Magiclamp uses Boost 1.57 which can be found at:

*   Windows:
    *   Includes in W:\pimdev\analytics\magiclamp\thirdparty\python2.7\build\boost\include
    *   Libs in W:\pimdev\analytics\magiclamp\thirdparty\python2.7\windows
*   Linux
    *   Includes in /appl/pimdev/analytics/magiclamp/thirdparty/python2.7/build/boost/include
    *   Libs in /appl/pimdev/analytics/magiclamp/thirdparty/python2.7/linux

</div>

</div>

</div>

<div id="footer" role="contentinfo">

<section class="footer-body">

Document generated by Confluence on Mar 06, 2019 11:45

<div id="footer-logo">[Atlassian](http://www.atlassian.com/)</div>

</section>

</div>

</div>