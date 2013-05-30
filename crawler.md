Crawler_Arch 
==============

Type : _search form_

Note1: In the context of this text, a 'relevant' element is one that has an attribute value in the same lexical 
field as the target eg. When looking for a search form, {'Book now', 'Tickets', 'Price list', 'Search', etc.} 
is a relevant list of values to look for in a `<table>`'s attributes.

Note2: 'Variable' refers to the element of a trip being studied. Eg. origin, destination, price, etc.

Note3: 'Value' refers to the value a variable can take in a specific trip. eg. Montreal, Toronto, 25$, etc.

NLP projects: Apache Stanbol(Java), nltk (python - _not sure_ )

Step 1 :Fill input fields of the search tool and submit (TODO -- Compart in clear patterns)
--------------------------------------------------------

**Part 1 : Hints to find form inputs**

	Method signature: def get_form_inputs()
	
	Goal: Find website's search form and returns the css selector and input space of each input fields

	Inputs:
			home_url	:	url of the homepage of the company's website
	Outputs:
			(an absence of ID means there is no such field or it should be left to default)

			navigation_log:			list of (previous_url,new_url,request) to get to form

			onewayReturn_list:		list of all oneway/return alternatives (2 here)
			onewayReturnID: 		css selector for the oneway/twoway field in the form
			origin_list:			All cities serviced by the cie
			originID:			css selector for the origin field in the form
			destination_list:		list of all destinations serviced by the cie
			destinationID:			css selector for the destination field in the form
			time_list:			list of all departure times
			timeID:				css selector for the time field in the form
			date_list:			list of all dates that must be surveyed.  
							(Can include month & year depending on format)
			dateID:				css selector for the date field in the form
			month_list:			list of all months that must be surveyed
			monthID:			css selector for the month field in the form
			year_list:			list of all years that must be surveyed.
			yearID:				css selector for the year field in the form
			traveller_type_list:		list of all traveller type allowed by the cie
			travellerTypeID:		css selector for the travellerType field in the form
			traveller_class_list:		list of all travel class offered by the cie
			travellerClassID:		css selector for the travel calss field in the form


Programmer assisted workflow:

*  Input the home_url
*  Prompt user to navigate to the page of the search from, recording context throughout (default: simple url)
*  Append list of context to navigation_log
*  Prompt user for search field ID's
*  Append to respective output variables(i.e. xxxID)
*  Prompt user for search field input space(i.e. what set of values could inputed here)
*  Append to respective output variable lists(xxxxx_list)

Automation version: (_Hints_)

*  Navigation log:
	1.  Get to home page
		1. Note header/status/content of page/resources and create a naviagationLog
	2.  Look for search form on the page. Try :
		1. Look for code `<input tabindex = x>` (trickshot)
		2. Get all tables
			1. Check relevance of parent `<div>`
			2. Check relevance of `<inputs>` found amongst children
			3. Check relevance of html comments above tables
			4. Choose table with highest utility ( come back to this step is fails later)
		3. (last resort) Get all `<input>` fields
			1. Check relevance of attributes
			2. Check parent relevance
			3. If needed, recurse back up to parent tree until `<hx>` tag
			4. Choose the best fields and try to find a common parent.
	3. If no form, table or other relevant element on page,
		1. Look for relevant links to other pages
		2. Visit
		3. Goto  '2. Look for search form on the page'
	4. Create temp variable table_selector,a css selector of the most valid parent element of all those `<inputs>`.
	
*  Input field id's
	1. Traverse input fields found in table pointed by table_selector
	2. Obtain identity(orgin, destination, date, etc.) of the field through its attributes 
		(name is pretty good with a lil' parsing)
	3. Identify the input fields with css selector
	4. Validate the identification has no duplicates in the page
	    1.  Assert len(document.findAll(selector)) == 1
*  Input space lists:
	1. Observe tag name
		*  `<select>` often has `<option>`'s under it
	2. `<inputs>` may call scripts that populate dropdown `<div>`
		*  Access those scripts
	3. Some text fields have a search button (eg. search for origin city)
		1.  Click that button
		2.  Enter 'a' in the field
		3.  Parse all entries generated (i.e. all entries staring with 'a')
		4.  Repeat for B-to-Z
	4. Look for quick wins:
		*  type = radio is usually a oneway/Return field
		*  `<input tabindex= x>` (trickshot) is a clear win
		*  (more to come...)


**Part 2 : Submit all possible permutation of the input space**

	Method signature:	def submit_form()
	
	Goal: Submit all possible permutation of input space in the search form and return the url 
	     and context of the next page, as well as weither or not it contains a final timetable

	Inputs:
			(an absence of ID means there is no such field or it should be left to default)

			navigation_log		:	list of (previous_url,new_url,request) to get to form

			onewayReturn_list:		list of all oneway/return alternatives (2 here)
			onewayReturnID: 		css selector for the oneway/twoway field in the form
			origin_list:			All cities serviced by the cie
			originID:			css selector for the origin field in the form
			destination_list:		list of all destinations serviced by the cie
			destinationID:			css selector for the destination field in the form
			time_list:			list of all departure times
			timeID:				css selector for the time field in the form
			date_list:			list of all dates that must be surveyed.
							(Can include month & year depending on format)
			dateID:				css selector for the date field in the form
			month_list:			list of all months that must be surveyed
			monthID:			css selector for the month field in the form
			year_list:			list of all years that must be surveyed.
			yearID:				css selector for the year field in the form
			traveller_type_list:		list of all traveller type allowed by the cie
			travellerTypeID:		css selector for the travellerType field in the form
			traveller_class_list:		list of all travel class offered by the cie
			travellerClassID:		css selector for the travel calss field in the form



	Outputs: 	List of tuples containing :
				navigationLog:  List of (this_url,next_url,request)
				is_timetable:	boolean; 
					returns True if form submission leads to a page containing a timetable
					false if the next state requires further action to reach the timetable page

*  For all permutations of all {input}List :
	1. Fill form using the inputID's
	2. Submit
	3. Return 
		*  Naviagation log (this_url, next_url, request)
		*  is_timetable, True if there is a final timetable(i.e. if there is a {$,£,¥, etc} 
		sign on the page AND 'Total')


Step 2 [intermediate] :Choose a trip alternative
----------------------------------------------------------


**Part1 : Hints to automate inputs generation**

	Method signature: get_trip_options_selector()
	
	Goal  : Find the web element(s) responsible for trip alternative selection 
		(eg. trip departure time on a given day, travel class, etc.)

	Inputs : 	
			navigationLog : 	List of (previous_url,new_url,request) to get to this page
			is_timetable :		False if intermediate step (non-terminal timetable)
			selection_pattern :	State the pattern used to fill trip_selectors
			

	Outputs : 	navigationLog : 	List of (previous_url,new_url,request) to get to this page
			is_timetable:		False if intermediate step (non-terminal timetable)
			selection_pattern:	States the pattern used to fill trip_selectors
			options_selectors:	List of css selectors pointing to trip elements to be chosen by user
						eg. [[option1_alternative1,option1_alternative2],
						[option2_alternative1,option2_alternative2,]]]
						-->  [[price_alt1, price_alt2],[hour_alt1, hour_alt2]]

*  navigationLog : 	List of (previous_url,new_url,request) to get to this page
	*  _simply pass down input_
*  is_timetable:		False if intermediate step (non-terminal timetable)
	*  _simply pass down input_
*  selection_pattern
	*  _simply pass down input_

*  trip_selector:
	*  Pattern1 : 
		*  Find list of radio buttons under a common parent with relevant name(i.e. under same column)
	*  Pattern2 :
		*  Radio buttons in the same row are usually for same trip, different fare 
		   (i.e. web price,refundable,etc.)
	*  Pattern3 : 
		*  Find a 'Add to cart' text field and click the `<href>`
	*  Pattern 4 :
		* Pattern 1 and 2 on same page
	*  Pattern 5 :
		*  Pattern 1 & 3 on same page  

**Part2 : Visit all possible trip alternatives**

	Method signature: branch_all_trip_options()
	
	Goal  : Create branches for different departure/arrival times,  alternatives.

	Inputs : 
			navigation_log : 	List of (previous_url,new_url,request) to get to this page
			is_timetable:		False if intermediate step (non-terminal timetable)
			selection_pattern:	States the pattern used to fill trip_selectors
			options_selectors:	List of css selectors pointing to trip elements to be chosen by user
						eg. [[option1_alternative1,option1_alternative2],
						[option2_alternative1,option2_alternative2,]]]
						-->  [[price_alt1, price_alt2],[hour_alt1, hour_alt2]]

	Outputs : 	trip_alternatives: List of tuples:
						navigationLog : updated navigation log with present request to next page
						is_timetable:	boolean; 
								returns True if form submission leads to a page
								containing a final timetable
								false if the next state requires further action 
								to reach the timetable page
				
					for each alternative trip option on the page

	
*  if is_timetable : pass
*  else:
	*  for each trip_selectors[choice]
		1.  Click first list selector element and  submit
		2.  Append an updated navigationLog
		3.  Append is_timetable (i.e. is there a {$,£,¥, etc} sign on the page AND 'Total') 
			#NOTE: bad heuristic...
		4.  Repeat for all selectors in the list


**Part3: Draw a utility function for the trip alternative selection procedure**

	Method signature: calc_utility()
	
	Goal: Determine which pattern lead to the highest number of distinct trip cases
	
	Input:
			next_page_list 	: List of trip_alternatives (for each pattern) 
	
	Output: 
			utility : number of different trip alternatives genrated by the pattern
_TODO_
 

Step 3 :Parse final schedule and price  
--------------------------------------
class ScheduleParser()
---------------------------
---------------------------

Note: The following steps should be performed with 
subclasses of ScheduleParser implementing the different parsing pattern available. 
Their ensuing utilities should then be compared for a final choice of pattern to
use on target pafe
	
**Part1 : Hints to generate parser settings (based on pattern)**
	
	"""
	Method signature:  def init()
	
	Goal  : Generates the parser's attributes according to a specific pattern

	Input: 
			navigation_log : 	List of contexts (previous_url,new_url,request) to get to this page
			parsing_pattern:	States the pattern (i.e. function) to used to aquire the data
			

	Output : 
			navigationLog : same naviagation log with url and request needed to get here.
			table_selectors : 	List of css selector(s) of the parent element containing the timetable(s)
			ext_data_selectors:	List of css selector(s) to external resource 
						(i.e. not all info is wihtin the table ; refer to mask for semantic)
			variable_masks:		List of dic used as a mask to interpret information obtained in 
						each table of table_selectors (i.e. headerValue : 
						busbudInterpretation/format)
				(eg. {
					'Fare' : price,
					'Service': company,
					'Depart' : (location, (day,date,month,year, hour)),
					'Arrives': (location, (day,date,month,year, hour)),
					'Duration': 'discard'  #To ignore this entry
					
				})
			ext_data_masks:		List used to interpret data obtained from the external resource fields
				(eg. [price,(Class, origin)] would mean the first entry in ext_data_selectors points 
				to the price of the trip, and the second entry points to a text fields containing both 
				the class of the trip and its origin)
			pasring_pattern :	states the pattern used to generate the above outputs
	"""	
	
**Part2 : Convert pattern-based settings into parsed tables**

	"""
	Method signature:  def run()
	
	Goal  : Extract data from the schedule timetable according given parsing_patterns

	Input: 
			navigation_log : 	List of (previous_url,new_url,request) to get to this page
			table_selectors : 	List of css selector(s) of the parent element containing the timetable(s)
			ext_data_selectors:	List of css selector(s) to external resource 
				(i.e. not all info is within the table ; action depends on parsing_pattern)
			variable_masks:		List of dic used as a mask to interpret information obtained in 
				each table of table_selectors (i.e. headerValue : busbudInterpretation/format)
				(eg. {
					'Fare' : price,
					'Service': company,
					'Depart' : (location, (day,date,month,year, hour)),
					'Arrives': (location, (day,date,month,year, hour)),
					'Duration': 'discard'  #To ignore this entry
					
				})
			ext_data_masks:		List used to interpret data obtained from the external resource fields
				(eg. [price,(Class, origin)] would mean the first entry in ext_data_selectors points 
				to the price of the trip, and the second entry points to a text fields containing both 
				the class of the trip and its origin)
			parsing_pattern :	States the pattern used to produce these inputs


	Output : 
			navigationLog : updated naviagation log with url and request needed to get here.
			parsing_pattern : States the pattern used to produce these outputs
			parsed_tables	: List of all ParsedTable objects in the page. Each objects contains:
				trips	: list of trips parsed in a ParsedTable. Each trip contains: 
				(variable: value,)
							{
							oneWay/return:,
							origin:, 
							destination:, 
							departTime:, 
							arrivalTime:, 
							travellerType:,
							travellerClass:,
							service:, 
							price:,
							}

	"""

*  Pattern 1 : Classic table w/ horizontal headers 
	*  `<table id= table_id>`
		*  `<thead> `
			*  `<tr>`
				*  `<th>Fare</th>`#Variable1
				*  `<th>Service</th>`#Variable2
				*  `...`
			*  `</tr>`
		*  `</thead>`
		*  `<tbody>`
			*  `<tr> </tr>` # Trip 1
			*  `<tr> </tr>` # Trip 2
			*  `...`
			*  `<tr>` #Trip n
				*  `<td>50$<td>`#value1
				*  `<td>GX40<td>`#value2
				*  ...
			*  `</tr>`
			*  `...`
			*  `<tr></tr>`
		*  `</tbody>`
	*  `</table>`

	def init()
	
	*  navigation log : 
		*  _idem as input_
	*  table_selectors: 
		1.  Pick tables with `<thead>` and/or `<th>` in it
		2.  Append a css selector unique to each table OR unique to this group of table
	*  parsing_patterns :
		*  _idem as input_
	*  variable_masks : (dic)
		*  dic keys are found in `<th>`'s
		*  dic values are reverse looked up in a lexical domain re table  
			*  eg. 'Fare' is interpreted as part of 'price' lexical field
	*  ext_data_selector :
		*  if 'price' is not in variable_mask
			*  return the css selector of the `<table>`,`<ul>`, `<div>` that contains 
			   `<input>` or `<select>` (i.e. a search form)
	*  ext_data_masks : 
		*  _(None)_
	
	def run()

	*  Access tables pointed by table_selectors
	*  Variables are looked up in variable_mask ' values field to interpret the information to 
		come (e.g. variable_mask['Fare']= price --> price column)
	*  Each `<tr>` of `<tbody>` is a different trip 
	*  Values in a row are recorded following the same order as the one found in variable_mask 
		(eg. '50$' is TripN's value for Variable1(i.e. price)) 
	
*  Pattern 2 : 'Headerless' horizontal table (aka the Communist)
	*  _(Similar to Pattern 1)_
	*  No `<thead>` or `<th>` 
	
	def init()

	*  navigation log : 
		*  _idem as input_
	*  table_selectors: 
		*  Get all tables
		*  _(alternate)_ Discard invalid tables right off the bat (via NLP)
			1. Check relevance of table
			2. Check relevance of parent `<div>`
			3. Check relevance of html comments above tables
			4. If needed, recurse back up the parent tree until `<hx>` tag and 
				evaluate relevance (eg. 'Bus Timetable', 'Confirm', 'Schedule', 'Fares')
			5. Keep tables with high enough utility  
		*  Append a  css selector unique to each table OR unique to this group of tables
	*  parsing_patterns :
		*  _idem as input_
	*  variable_masks : (dic)
		*  dic keys are found in the children elements of the first `<tr>`
		*  dic values are looked up in an re table of relevant words (eg. 'Fare' is interpreted as 
			part of 'price' lexical field
	*  ext_data_selector :
		*  if 'price is not in variable_mask
			*  return the css selector of the `<table>`,`<ul>`, `<div>` that contains 
				`<input>` or `<select>` (i.e. a search form)	
	*  ext_data_masks : 
		*  _(None)_
	
	def run()

	*  Simply skip the first `<tr>` and then start parsing each following `<tr>` as in pattern 1

*  Pattern 3 :Table-ception (a table within a table)
	*  `<table id= table_id>`
	*  `<tbody> `
		*  `<tr>`
			*  `<td>Fare</th>`#Variable1
			*  `<td>Service</th>`#Variable2
			*  `...`
		*  `</tr>`
		*  `<tr>`
			*  `<td>`
				*  `<table>`
					*  `<tbody>`
						*  `<tr>`...`</tr>`#Trip 1
						*  `<tr>`...`</tr>`#Trip 2
						*  `<tr>`...`</tr>`
						*  `<tr>`...`</tr>`
					*  `</tbody>`
			* `</td>`
		*  `</tr>`
	*  `</tbody>`
	*  _(Similar to Pattern1)_ 

	def init()

	*  navigation Log: _idem as input_
	*  table_selectors :
		*  _idem as Pattern 2_
	*  parsing_patterns:
		*  self.__name_
	*  variable_masks: (dic)
		*  dic Keys are found in the preceding sibling of the row (`<tr>`) holding leaf tables
		*  dic Values are the result of a lookup of the keys in an re table of relevant words 
			(eg. 'Fare' is interpretted as 'price')		
	*  ext_data_selector :
		*  if 'price' is not in variable_mask
			*  return the css selector of the `<table>`,`<ul>`, `<div>` that contains 
				`<input>` or `<select>` (i.e. a search form)	
	*  ext_data_masks:
		*  _none_
	  
	def run()

	*  if `<table>` pointed by table_selectors is leaf table(i.e. no children table), discard.
	*  else, find the leaf `<table>` and parse each `<tr>` as a trip in pattern 1

*  Pattern 4 :Table-ception 2 (a table within a table)
	*  `<table id= table_id>`
	*  `<tbody> `
		*  `<tr>`
			*  `<td>...</td>`#Contains table 1
			*  `<td>...</td>`#Contains two elements (variable, value)
			*  `<td>...</td>`#Contains table 2
		
	*  `</tbody>`
	*  _(Similar to Pattern1)_ 

	def init()

	*  navigation Log: 
		*  _idem as input_
	*  table_selectors :
		*  Select all tables using css selectors
	*  parsing_patterns:
		*  _idem as input_
	*  variable_masks: (dic)
		*  Get table X 's variable mask's keys the same way as pattern1 or pattern2
		*  Get the singleton's variable mask by interpreting the first `<td>` as a dic key
		*  Dic values are the result of a lookup of the keys in an re table of relevant words 
			(eg. 'Fare' is interpretted as 'price')
	*  ext_data_selector :
		*  if 'price' is not in variable_mask
			*  return the css selector of the `<table>`,`<ul>`, `<div>` that contains 
				`<input>` or `<select>` (i.e. a search form)	
	*  ext_data_masks:
		*  _none_
	
	def run()

	*  if `<table>` pointed by table_selectors is leaf table(i.e. no children table), discard.
	*  else, find the leaf `<table>`'s and parse them as in pattern1 or pattern 2
	*  The singleton's second `<td>` is a value interpreted according to the singleton's mask

*  Pattern 5 : `<ul>` table : implied headers i.e. Headers not stated anywhere on the page
	
	def init()

	*  navigation Log: 
		*  _idem as input_
	*  table_selectors :
		*  Append a distinct css selector for each `<ul>` in the page
		*  _(alternate)_ Discard outlier `<ul>` right off the bat (via NLP)
			1. Check relevance of `<ul>`
			2. Check relevance of parent `<div>`
			3. Check relevance of html comments above elements
			4. If needed, recurse back up the parent tree until `<hx>` tag and evaluate relevance 
				(eg. 'Bus Timetable', 'Confirm', 'Schedule', 'Fares')
			5. Keep tables with high enough utility  
	*  parsing_patterns:
		*  _idem as input_
	*  variable_masks: (dic)
		*  Keys == Value
		*  Values are infered from the data found in the table (using NLPand re)
			*  eg. Script should understand that anything with '$' is a 'price'
			*  eg. Use NLP library to identify 'Mexico' as an 'origin' or a 'destination'
	*  ext_data_selector :
		*  if 'price is not in variable_mask
			*  return the css selector of the `<table>`,`<ul>`, `<div>` 
				that contains `<input>` or `<select>` (i.e. a search form)	
	*  ext_data_masks:
		*  _none_
	
	def run()

	*  Variables are obtained from the values in variable_masks
	*  Each `<li>` is a trip
	*  Comma seperate content of `<li>` (following the appropriate variable_masks entry)
	*  Values in a row are recorded following the same order as the one stated in variable_mask
*  Pattern 6 : `<ul>` table :vertical headers 
	*  i.e. Each `<li>` contains both the variable and the value within the same string
	*  eg. 
		*  `<li> <p>Journey :Bolton/Horwich, Horwich Parkway Train Station to London, 
			Victoria Coach Station </p></li>`
		*  `<li> <p>Date : Saturday, 25 May 2013</p></li>`
		*  etc...
	
	def init()

	*  navigation Log: _idem as input_
	*  table_selectors :
		*  Append a distinct css selector for each `<ul>` in the page
		*  _(alternate)_ Discard outlier `<ul>` right off the bat (via NLP)
			1. Check relevance of `<ul>`
			2. Check relevance of parent `<div>`
			3. Check relevance of html comments above elements
			4. If needed, recurse back up the parent tree until `<hx>` tag and evaluate 
				relevance (eg. 'Bus Timetable', 'Confirm', 'Schedule', 'Fares')
			5. Keep tables with high enough utility  
	*  parsing_patterns:
		*  self.__name_
	*  variable_masks: (dic)
		*  Keys are the strings coming before the delimiter on each row. Delimiter could be {':','-','|',etc.}
		*  Values are the result of a lookup of the keys in an re table of relevant words 
			(eg. 'Fare' is interpretted as 'price')		
	*  ext_data_selector :
		*  if 'price is not in variable_mask
			*  return the css selector of the `<table>`,`<ul>`, `<div>` that contains 
				`<input>` or `<select>` (i.e. a search form)	
	*  ext_data_masks:
		*  _none_
	
	def run()

	*  Get the `<ul>` pointed by table_selectors
	*  Read each <li> under it
	*  The text fnound after a special character {':','-','|',etc.} is interpreted as data according 
		to variable_mask

*  Pattern 7-11 : Pattern 1 to 5 with a vertical headers
	*  Each `<tr>` of a table contains `<td> variable</td><td>value</td>`

	def init()

	*  Same as Pattern 1 to 5 except:
		*  variable_masks:  (dic)
			*  Keys are found every second `<td>`
			*  Values are the result of a lookup of the keys in an re table of relevant words 
				(eg. 'Fare' is interpretted as 'price')	
	
	def run()

	*  Same as pattern 1 to 5 except:
		*  trips values are found every second `<td>` in the tables (skipping the first `<td>` since
			it's a variable)
	
*  Pattern 12-16 : Patterns 1 to 5 with implicit headers
	*  Each 'table' has no explicit header mentioned on the page (i.e. 'origin', 'destination', etc.)

	def init()

	*  variable_masks' keys and values must be infered:
		1.  Generate all possible variable{origin, destination, price, depart date, etc.} 
			permutations with respect to the columns **OR**
		2.  Use NLP/re on data to infer the implicit variables that defines them
	
	def run()

	*  Same as Patterns 1 to 5	

**Part 3: Utility evaluation** 

	Method signature: def calc_pattern_utility()
	
	Goal : Caculate a performance utility for a parsed table
	
	Inputs:
			parsing_pattern : States the pattern used to produce these outputs
			parsed_tables	: List of all ParsedTable objects. Each objects contains:
				trips		: list of trips parsed from a web table. Each trip contains:
							{
							oneWay/return:,
							origin:, 
							destination:, 
							departTime:, 
							arrivalTime:, 
							travellerType:,
							travellerClass:,
							service:, 
							price:,
							}
		
	
	Outputs:
		table_utilities		: List of utilities acessing the performance of a parsing pattern on a table.
		is_continuous_tables	: Boolean: True if the multiple tables within a page are the continuation of 
			each other (i.e. bus stop time of passge spanning many tables)

_(in development)_

*  Options 1: RE/NLP based
	*  Use regular expressions and/or natural language processing to evaluate if a value(eg. 50$) is associated 
		to a proper variable(eg. 'price')
		*  eg.
			*	{ price: [20$, 10$, 'to Montreal', 45$, 15$],
			*	  destination: ['Quebec','Montreal', 5$, 'Toronto', 'Ottawa'],
			*	}
			*  	'to Montreal' doesn't match regular expression /^([1-9][0-9]*|0)(\.[0-9]{2})?$/
			*  	Similarly, '5$' doesn't match a city found in any database (alt. is not a string)
			*	This trip would get a 0.8 rating 20% of the information is wrong
	*  +1 for proper trip element association, 0 for inproper association
	*  Take the average of all items of each trips in a single table
	*  Append to table_utilities
*  Option 2: Programmer Veto
	*  The programmer can be prompted to binary rate the performance of the pattern on each table found in parsed_tables


Pattern summary
--------------
--------------

1.  Main Info Container : `<table>` vs `<ul>` vs `<div>`
2.  Header orientation : horizontal vs. vertical
3.  Header expliciteness : obvious (`<thead>`) vs explicit (first `<tr>` of main container) vs. 
	implicit (infered from data)
4.  External data : location of data outside main container and interpretation of the information found 
	at said location 
5.  Multiple tables per page : independent tables vs. tables' data is coninuous (eg. times for a bus stop 
	spanning many tables)

Interesting information to analyse once the *best* parsed data is chosen:
*  is_bilateral_price,  	# boolean ,to_fare = from_fare
*  is_surcharge, 		# boolean total price > to+from trips 

Usefull function:
*  format_time: Given all time data of a table, infers if {12hrs or 24hrs} and {1212 or 12:12}
