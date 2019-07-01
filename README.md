<h1>Treatstock API</h>


<h2>Introduction</h2>

Using treatstock api you can upload 3d model, get model size and weight, calculate printing price, make order.

Simplest using way it is creating printable pack using api, and then redirect client using treatstock.api url.

At first you should clone this project from git. In "examples" folder you can try start examples: 01_CreatePrintablePack.php.

All api requests making through "TreatstockApiService" class.  
Class constructor have parameter $privateKey - this is your personal company key for api access to treatstock. 
You should contact support@treatstock.com for get this privateKey.

<h2>Create Printable Pack</h2>

Creating instance of this class:

<pre>
$apiService = new \treatstock\api\v2\TreatstockApiService($privateKey);
</pre>

You can set debug mode to output all http request info:
<pre>
$apiService->setDebugMode(true);
</pre>

Let's make simple create printable pack request. It should contain files path and possible client location, default is US.

<pre>
$createRequest = new \treatstock\api\v2\models\requests\CreatePrintablePackRequest();
$createRequest->filePaths[] = './test.stl';
$createRequest->locationCountryIso = 'US'; // Optional params: to get printable pack price info
echo "\nSend create printable pack request...";
$createResponse = $apiService->createPrintablePack($createRequest);
echo "\nCreate printable pack response:\n";
echo \treatstock\api\v2\helpers\FormattedJson::encode($createResponse);
</pre>

Output example:
<pre>
Send create printable pack request...
Create printable pack response:
{
    "id": 200010,
    "redir": "https://www.treatstock.com/catalog/model3d/preload-printable-pack?packPublicToken=ee13f4e-4873bd7-424b569",
    "widgetUrl": "https://www.treatstock.com/api/v2/printable-pack-widget/?apiPrintablePackToken=ee13f4e-4873bd7-424b569",
    "widgetHtml": "<!-- ApiWidget: ee13f4e-4873bd7-424b569 --><link href=\"https://www.treatstock.com/css/embed-user.css\" rel=\"stylesheet\" /><iframe class=\"ts-embed-userwidget\" width=\"100%\" height=\"650px\" src=\"https://www.treatstock.com/api/v2/printable-pack-widget/?apiPrintablePackToken=ee13f4e-4873bd7-424b569\" frameborder=\"0\"></iframe>",
    "parts": [
        {
            "uid": "MP:1609949",
            "name": "test.stl",
            "qty": 1,
            "hash": "7e02f089e3e508459c967de27c10d45c",
            "size": null,
            "originalSize": null,
            "weight": null,
            "texture": null
        }
    ]
}
</pre>

<b>id</b>           - printable pack id<br>
<b>redir</b>        - url for page with printing model<br>
<b>widgetUrl</b>    - may be used to insert into iframe with custom settings<br>
<b>widgetHtml</b>   - ready to use iframe html code<br>
<b>parts</b>        - list of model3d parts info

Using information about parts info you can change it`s qty, change colors, get information about its possible weight.
You can simply upload you model, and get printing commission.

<h2>Colors and prices</h2>
If you want more control by your side, you can get list of prices in different colors and materials.
Example: 04_GetPrintablePackPrices.php

After you create printable pack, use printable pack id for prices request:
<pre>
// Try get printable pack prices
echo "\nGet printable pack prices...";
$pricesRequest = new \treatstock\api\v2\models\requests\GetPrintablePackPricesRequest();
$pricesRequest->printablePackId = $createResponse->id;
$pricesResponse = $apiService->getPrintablePackPrices($pricesRequest);
echo "\nGet printable pack prices response:\n";
</pre>

Output example:
<pre>
Get printable pack prices response:
{
    "pricesInfo": [
        {
            "printablePackId": 200037,
            "materialGroup": "PLA",
            "printer": "Bellair3D: Original Prusa i3 MK2",
            "providerId": 2998,
            "color": "Green",
            "price": 5.74,
            "url": "https://www.treatstock.com/model3d/preload-printable-pack?packPublicToken=cc849e5-ac24e04-fd2e25b&printerMaterialGroupId=16&printerColorId=1"
        },
        {
            "printablePackId": 200037,
            "materialGroup": "PLA",
            "printer": "Modern Blacksmith 3D Printing Services: Mytrix Dreamweaver 3735 Duo",
            "providerId": 941,
            "color": "Blue",
            "price": 5.99,
            "url": "https://www.treatstock.com/model3d/preload-printable-pack?packPublicToken=cc849e5-ac24e04-fd2e25b&printerMaterialGroupId=16&printerColorId=2"
        },
     ....
     ]
}  
</pre>

You can use url for printing directly in this color and this material with fixed printing price.

<h2>Make order</h2>

For partners with order flow we have api for creating orders.
Example: 05_PlaceOrder.php

You should set information about client location, phone, printer for manufacturing and 3dmodel materials/colors.

Output example: 
<pre>
{
    "orderId": 44649,
    "total": 10.24,
    "url": "http://ts.h3.tsdev.work/workbench/order/view/44649"
}</pre>

<h2>Make order in colors</h2>

For printing model3d in different colors use example: 06_PlaceOrderColored.php

You should use calls:
1. $apiService->createPrintablePack - create printable pack
2. $apiService->changePrintablePack - set manual colors and materials for current 3dmodel
3. $apiService->getPrintablePackOffers - get offers for printing. List of offers from differnt printing services.
Example output:
<pre>
{
    "modelTextureInfo": {
        "isOneMaterialForKit": false,
        "modelTexture": null,
        "partsMaterial": {
            "1536027": {
                "color": "Red",
                "materialGroup": "PLA",
                "material": null
            },
            "1536028": {
                "color": "Blue",
                "materialGroup": "PLA",
                "material": null
            }
        }
    },
    "offersList": [
        {
            "providerId": 934,
            "printPrice": 7.04
        },
        {
            "providerId": 3174,
            "printPrice": 8.39
        },
        {
            "providerId": 2421,
            "printPrice": 8.39
        },
        ....
    ]
}
</pre>

4. $apiService->placeOrder - place order using offer from previos request
Example output:
<pre>
{
    "orderId": 44650,
    "total": 7.04,
    "url": "http://ts.h3.tsdev.work/workbench/order/view/44650"
}
</pre>

<h2>Additions</h2>

* You can use "material" instead of "materialGroup" for printing directly in specified material.
* You can override "RequestProcessor" class for network or log control.

Example:
<pre>
class RequestProcessorExt extends \treatstock\api\v2\requestProcessor\RequestProcessor
{
    protected function debugWrite($params)
    {
       // Insert log into files
    }
}

$apiService->requestProcessor  = new RequestProcessorExt();
...
$createResponse = $apiService->createPrintablePack($createRequest);
</pre>

* Treatsock can autodetect client location by ip, use  
<pre>$createRequest->locationIp = '000.000.000.000';</pre>
instead of 
<pre>$createRequest->locationCountryIso = 'US'</pre>

* You can use zip files, and urls for big 3d models. Treatsock will download files from links.
<pre>
$createRequest->fileUrls[] = 'http://mysite.com/test.stl';
</pre>
or 
<pre>
$createRequest->fileUrls[] = 'http://mysite.com/test.zip';
</pre>
instead of 
<pre>
$createRequest->filePaths[] = './test.stl';
</pre>
