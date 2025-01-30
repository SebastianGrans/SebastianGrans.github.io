---
layout: post
title: Parts of my figure disappeared
---

<center>
<img src="{{ site.baseurl }}/images/20190722-Latex-pdf-issue/img.png" alt="Pinhole camera" style="display: block;"/>
</center>

<div markdown="1" style="display:block; background-color:#eff0f1; border-radius: 3px; padding: 0.001em 8px;">
**TL;DR:** Did you include a figure as a PDF in latex, and part of the figure is missing? Try converting it to PostScript using `pdf2ps`. 
</div>

I always try my best to use vector graphics when I write anything in Latex. If I am making plots, my pipeline is as follows

1. Make the plot in Matplotlib  
2. Save it as a PDF  
3. Tweak it in Illustrator
4. Save it as a PDF again

This time I was making the figure above, from scratch in Illustrator. As you can see, there are some regular raster images included into the vector graphic. As I included it in my latex document though, the image on the back of the box had disappeared. It was still visible when the file was viewed separately. 

When I googled this, most people had issues with fonts, which seem to be caused by the font not being embedded into the PDF. This didn't really help me, but thankfully I found a comment by [Norman Ramsey](https://stackoverflow.com/users/41661/norman-ramsey) in [this](https://stackoverflow.com/questions/2370864/problem-when-using-latex-includegraphics-with-some-pdf-files) thread, suggesting to convert the files to EPS (Encapsulated PostScript). I simply converted it to PS (PostScript), and that seemed to work just as well. For this I used `pdf2ps`. 

Hope this help someone. 

    
<details> 
  <summary>Some tags for the Google spider</summary>
    image disappeared from pdf latex  
    includegraphics pdf latex  
    part missing from graphics  
    missing figure from pdf  
    (I have no idea if these work)
</details> 
