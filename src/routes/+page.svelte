<script>
    import Sunburst from './Sunburst.svelte';
</script>

<main>
<h1>Imported Food in the United States of America</h1>
<Sunburst />
<div>
<h2>Welcome to a Zoomable Sunburst Chart!</h2>
<p>
    This chart displays how much the United States of America spends on imported foods.<br />
    <b>1.</b> Select a petal to zoom into your desired category. (You can even click on a subcategory!)<br />
    <b>2.</b> Select the center to zoom out of the subcategory.<br />
    <b>3.</b> Hover a wedge to see a subtotal of the subcategories.<br />
</p>
<h2>Writeup</h2>
<p><b>What is being displayed here?</b> In total, the U.S.A. spends $210,463,000,000 on imported food. Where does it come from? How much do we pay these countries?
    With my chart, I am displaying data from the <a href="https://www.ers.usda.gov/data-products/u-s-food-imports/" target="_blank">U.S.D.A. in 2023</a> that answers these questions.  
    The inner rings are the major categories. The outer rings are sub-categories. Clicking on an inner ring will reveal its sub-categories. 
    In this view, the cost and export breakdown are revealed slightly. If you click on a sub-category from here (or even from the home chart), it will take you to a detailed breakdown. 
    The wedge size like any pie chart represents the cost ratio out of a whole. By hovering a wedge, you can see how much each wedge costs. 
</p>
<p><b>Why was a zoomable sunburst chart chosen?</b> Earlier in this quarter, I found a zoomable sunburst chart on the <a href="https://observablehq.com/@d3/zoomable-sunburst" target="_blank">d3 website</a>. 
    My project at that time was about Pokémon and the data did not fit the chart so I left it alone. Eventually, I came back to it and designed my final project around it. 
    Implementing it was tricky at first, but I found a normal sunburst chart on the <a href="https://svelte.dev/playground/41124b6d1e1b417ea7d614d2d6299522?version=5.10.0" target="_blank">svelte website</a>. 
    By combining both templates, I was able to create this chart.
</p>
<p><b>Why not a pie chart or a bar chart?</b> I could have done a bar chart. Each bar would be the major category which would be subdivided into the sub-categories. 
    But a pie chart would be better since you can see a ratios/proportion a lot easier in a circle. 
    And then a sunburst chart is like a pie chart but allows sub-categories. And even further with a zoomable sunburst chart, you have a fun interaction available. 
</p>
<p><b>What were some challenges?</b> An issue I ran into was when you would enter a category, return to the home chart, and then enter the same category, the center image would not appear. 
    The reason why is because going home, just shrinks and fades out the image, it does not change the image. My image change function detects if the saved image link has change (which it did not going home), and then updates the image. 
    I created a global variable to store what page we are on, not the image link, and used that to determine if the image should change. <br />
    A second issue took embarrassingly long. How did I line up the center image? For the video, I hardcoded in the top and left CSS attributes to look good enough. 
    I tried creating a reference to the circle in the center that clicks to go home. It took me a while to grab the reference after the circle was created, but finding the center also proved nigh impossible. 
    Next, I grabbed the whole chart and tried to find the center of it. Just as difficult with the same results. Then I remembered: everything is in the same div tag. 
    I took the div tag and forced all of its elements to be centered. That's it.<br />
    A minor bit: the most tedious part was converting the U.S.D.A. data to a usable format. The zoomable sunburst chart takes in JSON data, but the raw data is formatted on multiple pages. 
    I used Google Colab to parse the data a write very short lines which I copy and pasted into the JSON file. 
</p>
<p><b>Why this topic?</b> As I mentioned, I picked my topic around using the chart. 
    So, I needed something with enough sub-categories and with proportions to warrant the use of this type of chart. 
    With the election season and Donald Trump winning, it made me think about borders and how much countries depend on each other for products. 
    I'm not trying to say "Oh, look how much we spend! We need to focus that money on in-country problems" or "See how much we lean on each other! We need to have everyone be reliant so we avoid war." 
    There's nothing grand about my designs. I just like food and all the cuisines I can eat. Whether the ingredients are sourced from their homeland or adapted recipes with local products, food is good.
</p>
<!-- <p><b>1. What have you done so far?</b> I have created a zoomable sunburst chart regarding how much the U.S.A. spends on importing food products. 
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
</p> -->
</div>
</main>

<style>
    main {
        text-align: center;
        padding: 20px;
    }

    h1, h2, p {
        font-family: Arial, Helvetica, sans-serif;
    }

    div {
        margin-top: 25px;
        padding: 5px;
        border-radius: 25px;
        background-color:#76E8CD;
    }

    h2 {
        padding-top: 5px;
        padding-bottom: 5px;
        border-radius: 25px;
        background-color:#88B8ED;
    }

    p {
        padding: 25px;
        border-radius: 25px;
        background-color:#A88CCC;
    }
</style>