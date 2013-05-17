Greyhound.com
==============

***Type*** : *search form *
---------------------------
---------------------------

Step1 :  Find page with search form 
-----------------------------------

1.  wow
    1.  Are
    2.  we
		1.  in?
    3.  and
		1.  in?
    4.  and

2.  out
3.  gunit
4.  pow


  	URL: home_page_url
	Context: none

	Goal: navigate to page with search form

	Return : 
			navigationLog : list of (previous_url,new_url,request) for visited pages.
			selector : selector for the search form parent element.


1.  Get to home page
    1. Note header/status/content of page/resources and create a naviagationLog
2.  Look for search form on page. Try :
    1. Look for <input tabindex = x> (trickshot)
    2. Get all tables
		1.  Check relevance of parent <div>
		2.  Check relevance of <inputs> found amongst children
		3.  Choose table with highest utility, coming back to this node if no data found
3.  (last resort) Get all <input> fields
    1. Check relevance of attributes
    2. Check parent relevance
    3. If needed, recurse back up to parent tree until <hx> tag
    4. Choose the best fields and try to find a common parent.


3. If no form, table or other relevant element on page,
	*1.3.1 Look for relevant links to other pages
	*1.3.2 Visit
	*1.3.3 Goto  1.2
4. Find a way to identify the input fields
	*1.4.1 id
	*1.4.2 name
	*1.4.3 fist element with class=name under parent element found above.

5. Validate the identification has no duplicates in the page
	*assert len(document.findAll(selector)) == 1*

Step 2: Inspect input fields for {input space}
----------------------------------------------
	"""
	Goal: obtain set of potential values each input can take.

	Param : where 'True' means the crawler must look for the field

		url					: url of page containing the search form 
		context 			: context required to get there
		tableSelector		: selector of table parent element 


		onewayReturn 		: boolean, field for one way vs. return
		onewayReturnFormat	: {radio button , dropdown menu, }
		origin 				: boolean, default = True
		destination 		: boolean, default = True
		locationFormat		: {dropdown, scriptGenerated}
		timeFormat			: ({
								(dayOfWeek),(date,month),(date,month,year),
								(time,date,month,year),(time)+(date)+(month)+(year), 
								(time)+(date)+(month-year), etc. 
								},
								{
								dropdown, calendarGenerated,etc.
								}
							  )
		departTime			: boolean, default = True  #General Moment, follows timeFormat
		returnTime			: boolean, tied to onewayReturn

		travellerType 		: boolean   #Age,social status,etc.
		travellerTypeFormat : {
								<select>, 
								seperate dropdown for # of passenger in each class,
								}, default = dropdown
		travelClass			: boolean
		travelClassFormat 	: {<select>, }, default = dropdown

		isDestFctOfOrigin	: boolean  # Are destination options = f(origin)

	Return : List/dic of sets of values each input can take,
			 dic of query selector that would allow the fields to be isolated easily
			 (id, name, etc.)

	"""
1. Traverse list of input fields found in table (or pseudo-table) of step1
	*2.1.1 Observe tag name
		<select> often has <option>'s under it
	*2.1.2 <inputs> may call scripts that populate dropdown <div>
		Access those scripts
	*2.1.3 look for quick wins:
		*type = radio is usually a oneway/Return field
		*<input tabindex= x> (trickshot) is a clear win
		* (more to come...)

Step 3 :Fill input fields of the search tool  and submit
--------------------------------------------------------

	"""
	Goal: Input all possible permutation of input in the search form 
		  to get solution space {price, time, cityRoute}

	Param:
			(an absence of ID means there is no such field or it should be left to default)

			navigationLog		:	list of (previous_url,new_url,request) to get to form

			onewayReturnList:	
			onewayReturnID: 
			originList:			All cities serviced by the cie
			originID:			selector for the origin field
			destinationList:
			destinationID:
			timeList:
			timeID:
			dateList:
			dateID:
			monthList:
			monthID:
			yearList:
			yearID:
			travellerType:
			travellerTypeID:
			travellerClass:
			travellerClassID:



	Return: 
			navigationLog: (this_url,next_url,request)

	"""

1. Fill and submit form with all permutations, and return navigation log
	
Step 4 :Choose an origin-destination trip to take that day
----------------------------------------------------------

	"""
	URL: "https://greyhound.com.au/Bookings/make-a-booking/bus-service-availability.aspx"
	Context: inputs {oneWay/return, origin, destination, departDate, return date, traveller's age}

	Goal  : Create branches for different departure/arrival times alternatives

	Param: 
			navigationLog : 	Min list of (previous_url,new_url,request) to get to this page
			trip_selector :		List of listed trip selectors (optional)

	Return : 
			navigationLog : updated naviagation log with present request to next page

			


	"""
1. If given a list of selectors pointing to choose a trip alternatives buttons,
	*click first list selector element, submit and update navigationLog
	*repeat for all selector in the list

2. Otherwise, find list of radio buttons under a common parent with relevant name
	*then go to 4.1

Step 5 :Choose a return trip to take 
------------------------------------

*(idem as step 4)*

Step 6 :Parse final schedule and price
--------------------------------------
	"""
	Goal  : Create branches for different departure/arrival times alternatives

	Param: 
			navigationLog : 	Min list of (previous_url,new_url,request) to get to this page
			trip_selector :		List of listed trip selectors (optional)

	Return : 
			navigationLog : updated naviagation log with url and request needed to get here.
			trips		  : list of dics for all possible trips. Contains:
							{
							oneWay/return,
							origin, 
							destination, 
							departTime, 
							arrivalTime, 
							travellerType,
							travellerClass,
							service, 
							price,
							is_location_time_format,#boolean, location and time of departure are in the same field
							locationFormat,			# Give an re (City name? Adress in city?)
							timeFormat, 			# Give an re (12 or 24 hr)
							is_unilateral_price,  	# boolean ,to_fare = from_fare
							is_surcharge, 			# boolean total price > to+from trips  
							
							}

	"""

