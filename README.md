# Dynatrace Demandware Fastpack


![images_community/download/attachments/215745785/icon.png](/images_community/download/attachments/215745785/icon.png)

The Dynatrace FastPack for Demandware provides a preconfigured Dynatrace profile custom tailored to Demandware environments. This FastPack contains sensors, a template system profile with measures and business transactions, and dashboards for the Demandware platform. Because the Demandware integration with UEM leverages CORS, there are a few measures that will likely have to be customized to your environment. 

##Customizations:
Dynatrace for Demandware is only for UEM via [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) as java agents are not allowed on the Demandware JVMs. As a result of this, some of the custom measures and business transactions in the fastpack are dependent User Action names and CSS Selectors specific to the Demandware supplied SiteGenesis store. Because the web frontend of a Demandware store can be customized, there's no guarantee the foundation of the measures will exist in your store.  However, these can easily be replaced.

###Customization #1:

- Username Tag

For the Username Tag, this fastpack leverages the CSS Selector ```[class='user-account']@title```, which is stored in a Page Action Metadata measure called Username.  The following Transformation regular expression is run on the result:

```
User: (?!Login \/  Register$)(.*)
```

In the case of SiteGenesis, when a User is not logged in, the value is filled in with 'Login / Register'. The regex used here excludes this state so that visits are not mis-tagged with the default value. If your store displays the username with a different CSS Selector, adjust the particulars in the corresponding sections.

Username UEM Configuration:
![Username UEM Configuration](/images_community/download/attachments/215745785/Username_UEM_Configuration.png)

Username Meta Measure Configuration
![Username Meta Measure](/images_community/download/attachments/215745785/Username_Meta_Measure.png)

To learn more about metadata measures, please refer to [Dynatrace documentation](https://community.dynatrace.com/community/display/DOCDT63/System+Profile+-+User+Experience#SystemProfile-UserExperience-WebSettings).

###Customization #2: 

- Conversion Goal

The Conversion Goal is a Page/User action used to identify when a user completes an order. Using a conversion goal has many benefits to understanding the conditions under which a user did or did not complete a purchase. 
In the Demandware Fastpack, we configured the Conversion Goal to be set to a measure called Place Order Count. Place Order Count is a User Action Count measure based on the Action Name:

```
click on "Place Order"
```

You can change your conversion goal by substituting any other measure you create.

Conversion Goal UEM:
![Conversion Goal UEM](/images_community/download/attachments/215745785/ConversionGoal_UEM.png)

Conversion Goal Measure
![Conversion Goal Measure](/images_community/download/attachments/215745785/ConversionGoal_Measure.png)

###Customization #3:

- Abandoned Cart
 
 The Abandoned Cart measure counts the number of Visits where users added at least one item to their cart but did not purchase. Along with counting how many abandoned carts there are in a given time period, the business transaction also tracks which pages customers are abandoning on.
 The Business Transaction is reliant on three measures:
- Add to Cart: a User Action Count method
- Non Conversion Goal: A conversion goal measure that tracks when users don't convert. This only works when the Conversion Goal is set as in Customization #2
- Visits - Exit Page - BT: a measure that captures the exit page of a visit

The only measure you may have to customize is Add to Cart, as it is based upon the button name of Add to Cart. The value the measure is looking for is an action name regular expression:

```
(.*)(?i)add to cart(.*)
```

The regular expression looks for a User Action containing 'add to cart' regardless of capitalization and what precedes or follows it.

Add To Cart Business Transaction Measure:
![Add To Cart BT](/images_community/download/attachments/215745785/AddToCartBT.png)


Find further information in the [dynaTrace community](https://community.dynatrace.com/community/display/DL/Demandware+FastPack). 




