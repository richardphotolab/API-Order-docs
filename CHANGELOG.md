<p align="center">
  <img width="140" height="102" src="https://gfs-na.richardphotolab.com/img/logo/rpl-logo.png">
</p>

# Documentation Changelog
+ Revisions 1.2.1 (2024/12/09)
	- Moved struct notes to their own column (`create` endpoint)
    - Added a note for `sourceImage` (`create` endpoint)
    - Revised copy & formatting in several notes (`create` endpoint)
    - Added warning note concerning error messages (`create` endpoint)
    - Smalls tweaks to one of the examples (`create` endpoint)
+ Revisions 1.2 (2024/04/11)
	- Added `mode` field to `created` and `shipped` endpoint responses
	- Added new [`TESTING`](TESTING.md) document to explain test mode
	- Updated `create` and `shipped` endpoints with copy/link to `TESTING` document
+ Revisions 1.1.9 (2024/01/23)
	- Tweaked some wording for uniqueId warning on `create` endpoint
	- Added `backPrintText1` and `backPrintText2` fields to `create` payload to support reverse printing (photo prints)
	- Changed emoticons to better communicate the style of message
	- Added explanation concerning empty payload values
	- Added explicit path to `created` and `shipped` endpoints
+ Revisions 1.1.8 (2023/10/14)
	- Added `text` field to payload to support an arbitrary string on line items
+ Revisions 1.1.7 (2023/07/27)
	- Updates to `shipped` endpoint to provide better description
	- Added brief process flow overview to `shipped` endpoint

+ Revisions 1.1.6 (2022/11/17)
	- Update datetime to properly show as ISO8601Zulu on `shipped` endpoint
	- Fixed the `create` payload not properly showing `address2` as optional

+ Revisions 1.1.5 (2022/10/26)
	- Modification of `crop` field in payload

+ Revisions 1.1.4 (2022/10/11)
	- Created new doc for order/shipping endpoint (with new layout)
	- Changed order/create endpoint to new layout
	- Tweaks across several pages

+ Revision 1.1.3 (2022/10/10)
	- Separate single page documentation into general and endpoint specific pieces
	- Modifications to order/create endpoint to make it jive better with new changes

+ Revision 1.1.2 (2022/09/16)
	- Fix item source image URL in example payload to a working version
	- Correct order number field name from `orderPONum` to `orderNumber`
	- Add Richard's internal identifier as `richardId` to order response payloads

+ Revision 1.1.1 (2022/05/06)
	- Add `uniqueId` data to response payload

+ Revision 1.1 (2022/05/28)
	- Add revision marker to document

+ Revision 1.0 (2022/05/28)
	- Switch to public visibility
