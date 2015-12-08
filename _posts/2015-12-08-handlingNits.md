---
layout: post
title: Handling Nits
---

We all have little nits that we find in a codebase.  During code reviews those nits may stand out, but are too minor to bring up.  Over time they can compound into a bigger issue for you.  Sometimes the easiest approach of opening a line of communication with the author can go a long way, "you know, X, we prefer to initilize our variables this way."  
Other times you may need to look past them especially when a suggestion has been made and ignored or worse turned the author defensive.
The third option is to look for a pattern and refactor the code.  In a past project I took a minor annoyance of comment defined behavior sections and turned it into a feature that helped us identify issues in production as soon as they happened.  We had a developer who made a habit of defining behavior sections in code using comments.  Check out this Java Spring controller:

```java
@RestController
@RequestMapping(value = ClientController.REQUEST_MAPPING)
@Api(value = "client", description = "CRUD client management.")
public class ClientController extends AbstractRestHandler {

  public static final String REQUEST_MAPPING = "/api/admin/client";

  private static final Logger log = LoggerFactory.getLogger(ClientController.class);

  @Autowired
  private IClientService clientService;

  @RequestMapping(value = "", method = RequestMethod.POST, consumes = { "application/json",
      "application/xml" }, produces = { "application/json", "application/xml" })
  @ResponseStatus(HttpStatus.CREATED)
  @ApiOperation(value = "Create a client resource.", notes = "Returns the URL of the new resource in the Location header.")
  public void createClient(@RequestBody Client client, HttpServletRequest request,
      HttpServletResponse response) {
    Client createdClient = clientService.create(client);
    response.setHeader("Location",
        request.getRequestURL().append("/").append(createdClient.getId()).toString());
  }

…

////////////////////// Exception Handling Here//////////////////////////

…

//////////////////////Validation HERE ///////////////////////////////////
}

```

Other than adding extra noise to the code, there is no way to guarantee the code will continue to be maintained according to the comments.  Let's say a new person is added to the team or you have a long class and don't scroll down far enough to see the comments.  Then as soon as the code is changed the comments are possibly incorrect.  

Plenty of IDE's can help pull out code into super classes, interfaces, or methods.  In STS to do this you can use the menu 

	Refactor → Extract Super Class
		       → Extract Interface
 		       → Extract Method

The benefit from his sorting the code by behavior gave me a chance to see many patterns in our code.   Beginning developers may not realize classes do not just encapsulate state, but behavior as well.  For this example I extracted the exception handing and validation into an Abstract Controller that all the other controllers extended and removed a good bit of duplicate code.  Then I remembered a suggestion from one of Joel Spolsky's books to automatically create defect issues on crashes.  We now had our error handling in one place, all I had to do was make sure we were in production when an issue was created, that it was not a duplicate of the same issue in the past 30 days, ignore exceptions we didn't care about, and use the API from Pivotal Tracker.    

We do not always get lucky about removing every nit we find in a codebase, but if you see this one of my particular irks see if you are able to pull out the code and change the comment into a class or method.  Check out Martin Fowler's book, <a rel="nofollow" href="http://www.amazon.com/gp/product/0201485672/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0201485672&linkCode=as2&tag=ansbl0f2-20&linkId=5P2U42QKHMWXERYP">Refactoring: Improving the Design of Existing Code</a><img src="http://ir-na.amazon-adsystem.com/e/ir?t=ansbl0f2-20&l=as2&o=1&a=0201485672" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" /> to see how you can better refactor problem areas in your code.
