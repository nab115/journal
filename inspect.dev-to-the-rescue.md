## January 23rd, 2023

## Debugging broken website on mobile

naranbabha.com was broken on mobile, and my guess was it was due to a JavaScript issue. The problem was that I coulnd't easily debug my website on mobile since I didn't have a macbook. However, I finally came across a very handy tool - Inspect.dev, which had a free trial that I downloaded. This allowed me to use dev tools *sans macbook* on any website I brought up in Safari, by connecting my iPhone to my PC via USB.

Since the website worked as expected on desktop, I assumed it was some sort of compatibility issue with the JavaScript code, perhaps the use of ES6 features like template strings weren't supported and that was causing the code to break on mobile. What I discovered is that the issue was with the ```assert``` keyword. To display my projects on the website, I have a JSON with each project's details such as title, description, etc. Using JS I dynamically create a project card for each item in the JSON. To bring the data into the JS file, I imported it via 

```
 import projects from '/projects.json' assert {type: 'json'};
```

However, it appeared that ```assert``` is an experimental feature that is not yet supported on mobile browsers.

So the alternative is to ```fetch``` the json file. I tried this initially at the top level with await (since fetch is async), which worked on desktop, but this was also giving an error on mobile. Perhaps mobile browsers don’t have support for await without async at the top level of a module.

So I then tried to place it in the fucntion passed to document.ready()


```
$(document).ready(async function() {
    const res = await fetch("/projects.json");
    const projects = await res.json();
    console.log("continuing with document ready");
    initial_jiggle();
    bounce($("#down_arrow"), 15, 750);
    fillProjects(projects);
});
```


But this didn’t work. 

https://stackoverflow.com/questions/69984178/async-await-with-jquery-document-ready

It appears that there might be an issue with document.ready not working with an async handler in jquery 3.2.1 (the version I was using). Also as an aside it looks like ```$(document).ready(handler)``` is deprecated, and simply ```$(handler)``` is recommended.

So there were two solutions :

1. Move fetch() to fillProjects() and make that function async
    ```
    async function fillProjects() {
        console.log("attempting to fill projects . . .");
        const res = await fetch("/projects.json");
        const projects = await res.json();

        projects.forEach((p) => { ... }
        ...
    ```

    This worked on desktop and copying it into the console using Inspect.dev it appeared to work on mobile.

2. Upgrading jquery

    This also worked after updating to jquery 3.6.3

Even though simply updating the jquery version fixed the issue, I decided to keep keep ```await fetch``` inside the fillProjects() function, since nothing else depends on the JSON fetch aside from that function.


