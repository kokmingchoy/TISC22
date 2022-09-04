# Level 4B

![image](https://user-images.githubusercontent.com/82754379/188318980-32cbdff8-6a92-4a03-8d34-ff9fd8a4d0d7.png)

Heading over to the given URL at https://d20whnyjsgpc34.cloudfront.net/ I saw a page with (six) cat pictures:

![image](https://user-images.githubusercontent.com/82754379/188319297-21c0eb57-3ce8-449e-b9f0-f32a6f7db61f.png)

The source code for the web page revealed what could be clues:
```html
    <!-- Passcode -->
    <h1 class="mb-3">Cats rule the world</h1>
    <!-- Passcode -->
    <!-- 
      ----- Completed -----
      * Configure CloudFront to use the bucket - palindromecloudynekos as the origin
      
      ----- TODO -----
      * Configure custom header referrer and enforce S3 bucket to only accept that particular header
      * Secure all object access
    -->
```

I also found it peculiar that the *alt* text for the images did not seem to bear any relation to the content of the images.
Perhaps there was another clue in the *alt* text?
```html
  <!-- Gallery -->
  <div class="row">
    <div class="col-lg-4 col-md-12 mb-4 mb-lg-0">
      <img src="img/photo3.jpg" class="w-100 shadow-1-strong rounded mb-4" alt="Boat on Calm Water" />

      <img src="img/photo1.jpg" class="w-100 shadow-1-strong rounded mb-4" alt="Wintry Mountain Landscape" />
    </div>

    <div class="col-lg-4 mb-4 mb-lg-0">
      <img src="img/photo2.jpg" class="w-100 shadow-1-strong rounded mb-4" alt="Mountains in the Clouds" />

      <img src="img/photo6.jpg" class="w-100 shadow-1-strong rounded mb-4" alt="Boat on Calm Water" />
    </div>

    <div class="col-lg-4 mb-4 mb-lg-0">
      <img src="img/photo5.jpg" class="w-100 shadow-1-strong rounded mb-4" alt="Waves at Sea" />

      <img src="img/photo4.jpg" class="w-100 shadow-1-strong rounded mb-4" alt="Yosemite National Park" />
    </div>
  </div>
  <!-- Gallery -->
```
