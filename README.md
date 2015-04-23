# Flow-Mark-Down
Flow Mark Down is a text based syntax for marking down flow diagrams

v .01

|Work in progress|

FMD allows you to use a structured, machine readable (hopefully!) syntax to create flow diagrams. 

It's meant to allow you to quickly prototype and iterate flows for your projects before you formalise them in to diagrams. 

It is hoped that in the future we will make parsers and tools to convert these flows in to diagrams. 

Documentation
=============

Each flow consists of a title and a series one or more flow sequences. If there is more than one flow sequence, line numbers must be used.


[Entity]
	e.g. [Name, Id] // Two separate entities
	e.g. [PersonEntity Id] //Person entity and/or a Person's id 

Entities can be defined in one location at the start of the sequence or group or they can be dynamically defined during a sequence (above)
	e.g. 
	[PersonEntity Id, Name, Age, Sex]

<Decision T,F> where T (optional),F (optional) is the number of the next flow in this sequence 
	e.g. <Has Registered> -> [User] - return user if registered
	e.g. <Has Registered 2> -> [User] - return user if registered, if not (so has failed) go to sequence 2
	e.g. <Has Registered 2,3> - if registered go to sequence 2, else go to sequence 3 
	Note: If no T or F listed, end flow if false. e.g. <Has Registered> -> [User] will not return user if F condition
	e.g. <Has Registered @needsregister> -> [User] - return needs register status else return the user

-> Flow direction or add or update
-x  Delete
>> Got to another flow sequence

(Database)
	e.g. [Person] -x (SomeDatabase) would delete a person 
	e.g. [Person] -> (SomeDatabase) would add or update a person
	e.g. [Person Id + PersonBelongings] -> (SomeDatabase [Person Id]) -> <Person Exists> -> (SomeDatabase PersonBelongings) //Write person belongings to the database if the person with id exists

@status or command or instruction
	e.g. @fail
	e.g. @exists
	e.g. @needsregistration
	e.g. @wait for push notification
	No need to state OK status

|Process|

Domain or object or thing (such as a server, device etc) have no delimiter. 

Overall Note: Single quotes can be used to further define a name or thing on that object  or to include extra non-structured data. 
	e.g. API 'Customer' would be the customer object on the API object
	e.g. (SomeDatabase [Member] 'tbl_member, tbl_settings') might be used to assist the reader to know some extra information about where to store things
 
//Comment about the thing that comes next



**Notes**
* Null, fail or error do not have to explicity handled. Consider the following flow: Client -> [Confirm Code] -> API -> (HighSpeedCache) -> |resend confirm email|. In this flow - Note we don't need to explicitly define that the flow will fail if the [Confirm Code] is not in the  (HighSpeedCache) database. The flow will stop. 
* Flows must start with a title, followed by one or more lines of flow sequences. If there is more than one item in the sequence, the lines must be numbered.

Examples
--------

Send person id to server, check HighSpeedCache database for it. If it doesn�t exist go to 2, otherwise go on to next step and update the device id for that person.

Set Person Device Id Flow //Flow title
	1. [Person Id + DeviceDetails] -> API -> (HighSpeedCache [Person Id]) -> <Person Exists 2> -> (HighSpeedCachet [DeviceDetails])
	2. @needsregistration -> [Person] -> API -> (HighSpeedCache) -> <1> //go to step 1 once registered

Delete Person Flow
	1. [Person Id] -> API -> (HighSpeedCache) -> <Exists> -x (HighSpeedCache) //delete the person with Id from HighSpeedCache if they exist. No need to define entities on the database as they can be inferred

Compress Image Process Flow
	1. [Image] -> API -> (Blob Storage) -> (Process Queue) -> |Resize and compress image| -> (Notification Queue) -> @imagecomplete_push


Real World Examples
-------------------


**User Flows**

**Login flow**
Client -> [User + Pass] -> API -> (SomeDatabase [Member]) -> [Access Token + Refresh Token] -> (' [Token]) -> [Token id] -> Client

**Refresh Token flow**
	1. Client -> <Token Expired> -> <Has refresh token 2> -> [Refresh Token] API -> <refresh token good> -> |Create tokens| -> [Access Token + Refresh Token] -> (HighSpeedCache) -> [Token id]
	2. @norefreshtoken >> Login Flow 

**Invalidate token flow**
API -> [Token] -x (HighSpeedCache) 

**Access using token flow**
Client -> [Token Id] -> API OWIN -> (HighSpeedCache) -> [Token] -> <Has token @401> -> |Setup authorised user| -> |perform intended process|

**Register user flow**
Client -> [Email Address + Pass] -> API -> [New User] -> (HighSpeedCache) -> [New User Confirm Code] -> |Send email to user|  >> Registration confirm user flow

**Registration confirm user flow**
Email client -> [Confirm Code] -> API -> (HighSpeedCache) -> <Confirm code good> >> Create Account Flow 

**Resend confirm email flow**
Client -> [Confirm Code] -> API -> (HighSpeedCache) -> |resend confirm email|

**Create account flow** 'no public API, must start from registration confirm user flow'
API-> [Confirm Code] -> (HighSpeedCache) -> [New User] -> (SomeDatabase [Member]) -> [User] -> (HighSpeedCache)

**Password reset initiate flow**
API -> |Generate reset code| -> [Confirm Code] -> (HighSpeedCache) -> |Send email to user with reset code| -> Client -> @display check email message

**Device password reset confirm flow**
Email client -> Client -> [Confirm Code + New Password] -> API -> (HighSpeedCache) -> <is code correct @try again> -> (SomeDatabase) -> [Member] -> |Set member password| -> (SomeDatabase) -> (HighSpeedCache) -> Client -> @Reset success message
 
**Logout user flow**
Client -> API -> [User Token] -x (HighSpeedCache)
