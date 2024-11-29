<script>
    import Sunburst from './Sunburst.svelte';
</script>

<main>
<h1>Imported Food in the United States of America</h1>
<Sunburst />
<h2>Welcome to a Zoomable Sunburst Chart!</h2>
<p>
    This chart displays how much the United States of America spends on imported foods.<br />
    <b>1.</b> Select a petal to zoom into your desired category. (You can even click on a subcategory!)<br />
    <b>2.</b> Select the center to zoom out of the subcategory.<br />
</p>
<h2>Writeup</h2>
<p><b>1. What have you done so far?</b> I have created a zoomable sunburst chart regarding how much the U.S.A. spends on importing food products. 
    When you click a food category, it zooms into that category to reveal more subcategories. 
    Then you can click a subcategory to reveal where the subcategory is imported from and how much it costs. 
    If you hover a "petal," it will show how much it costs in total with its subcategories. 
    If the center of the chart is clicked, then it will zoom out. 
    The center also houses the image which. PS the image will not align on all devices. It is hardcoded so I can use it for the video presentation.
</p>
<p><b>2. What will be the most challenging of your project to design and why?</b> Oddly the chart was the easiest. 
    I found a template for a static sunburst chart on the Svelte website and then adapted the zoomable variant from the D3 website. 
    Getting the images to switch to what I want was the hardest. There was an issue where if you leave a category and return, the image didn't appear. 
    This was because of how the subcategories work and how the main page has no image. I only call for a change in image if the previous image was different than the desired image. 
    So Main page to Fruit to Main to Meat works. But Main to Fruit to Main to Fruit does not. When returning to Main, the last image is still saved so I can do the fade out effect. 
    So I had to some how keep the image when fading out but also change it so it can be called back when returning.
    I set a global variable that changes to save what the current image should be. So even though the image doesn't change while fading out, global will say "Home."
    Then global is compared to tell if an image should be displayed.
    Now for the most tedious part? Adding the data. The chart requires a JSON file and how the data from USDA works does not mesh well. 
    So I used Google Colab to parse data and write very short lines to insert into the JSON file. Then copy and paste every entry which I guess I'm used to from my job.
</p>
</main>

<style>
    main {
        text-align: center;
        padding: 20px;
    }

    h1, h2, p {
        font-family: Arial, Helvetica, sans-serif;
    }
</style>