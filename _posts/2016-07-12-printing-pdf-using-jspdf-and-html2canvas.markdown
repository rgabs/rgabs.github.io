---
layout: post
title: "Printing the document from front end only using jsPDF, html2canvas"
date: 2016-07-12 00:34:00
author: Rahul Gaba
tags: "jsPDF, html2canvas, canvg, printing, browser, view"
---

##Why do I need this?
As a front end developer, we all come across a problem statement where the client wants to get the screenshot of front-end views in a single PDF.

Though we can print PDF by triggering browser's default print button functionality(similar to Ctrl+P) but that approach has some limitations, which are:
* Styling issues: The browser is programmed to print only the necessary info(text) when we do (Ctrl+P). The browser ignores some css properties box shadow/background.
* Working with SVG's: If you have a lot of charts on your page then it becomes a pain to write a separate [print specific CSS](https://css-tricks.com/css-tricks-finally-gets-a-print-stylesheet/).
* If you want to print multiple views: You cannot print multiple views in a single pdf file. You will need do some hack in your front-end router to merge all the pages in a single route and then trigger the print button. I tried it in angularJS and it is not something which I would recommend to do.

So if you think that you won't face the issues mentioned above, then you can go ahead and use [Ctrl + P](https://css-tricks.com/css-tricks-finally-gets-a-print-stylesheet/).

And if you think that you might face these issues, then this blog is for you.

##Initial Setup:
We will be using jsPDF for printing PDFs from the Client side. I believe it is a great tool but unfortunately it doesn't have a good documentation. So I am going to share how I managed to take exact similar printout of my view using jsPDF.

Here's the plugins that I used in my case with the versions. Please make sure to install the exact versions. Though if you want to do some experiment, please go ahead and try the new/beta versions and let me know if they work well:
* `Canvg(v1.0.0)` plugin: To convert SVG elements to canvas(Use this only if your view contains SVG elements). jsPDF isn't very good with SVG elements.
* `html2canvas(v0.5.0-beta4)`: To convert all our html view into a single canvas which will take care of maintaining the height and width of the view.
* `jsPDF(v1.0.272)`: For creating a blank PDF and writing html/images to PDF. Note that in our case, we will be writing canvas objects to the pdf which is a beta feature og jsPDF library. But it worked like a charm in both of my projects.

##The Real stuff:
Now, lets get started with some coding.

Firstly, we would need to replace all the svgs in out body to canvas Object. You don't need this if you do not have any SVG elements on your webpage.

```
//replace all svgs with a temp canvas
var replaceSVGwithCanvas = function(callback) {
  //find all svg elements in $container
  var $container = $('body');
  //$container is the jQuery object of the div that you need to convert to image. This div may contain highcharts along with other child divs, etc
  var svgElements = $container.find('svg');
  svgElements.each(function() {
    var canvas, xml;
    canvas = document.createElement("canvas");
    canvas.className = "screenShotTempCanvas";
    //convert SVG into a XML string
    xml = (new XMLSerializer()).serializeToString(this);
    // Removing the name space as IE throws an error
    xml = xml.replace(/xmlns=\"http:\/\/www\.w3\.org\/2000\/svg\"/, '');
    //draw the SVG onto a canvas
    canvg(canvas, xml);
    $(canvas).insertAfter(this);
    $(this).hide();
  });
  callback(); //to be called after the process in finished
};
```
After our SVG elements are replaced, we will use `html2canvas` plugin to convert the HTML view to a single canvas object.

```
replaceSVGwithCanvas(function onComplete() {
  html2canvas(document.body, {
    onrendered: function(canvasObj) {
      //save this object to the pdf
    }
  });
});
```

We will create a new jsPDF Object and configuration which will be passed while adding HTML.

```
var pdf = new jsPDF('l', 'pt', 'a4'), // landscape/point(Unit)/A4(size)
pdfConf = {
  pagesplit: false, //Adding page breaks manually using pdf.addPage();
  background: '#fff' //White Background.
};
```

<!-- function onBodyRender(canvasObj, defaultCompanyLabel, callback) {
      var pdf = new jsPDF('l', 'pt', 'a4'),
        pdfConf = {
          pagesplit: false,
          background: '#fff'
        };
      document.body.appendChild(canvasObj); //appendChild is required for html to add page in pdf
      pdf.addHTML(canvasObj, 0, 0, pdfConf, function() {
        document.body.removeChild(canvasObj);
        pdf.addPage();
        $('.right-container').show(); //show cards only for the second page
        html2canvas(document.getElementsByClassName('right-container')[0]).then(function(cardsCanvas) { //render only thr cards block and add them to a new page
          document.body.appendChild(cardsCanvas);
          pdf.addHTML(cardsCanvas, 20, 20, pdfConf, function() {
            document.body.removeChild(cardsCanvas);
            pdf.save(defaultCompanyLabel + '.pdf');
            $rootScope.showLoader = false;
            callback();
          });
        });
      });
    } -->
