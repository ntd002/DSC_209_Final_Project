<script>
    import {onMount} from 'svelte';
	import * as d3 from 'd3';

	let element;

    onMount(async function() {
            //grab json data
		let data = await d3.json('https://raw.githubusercontent.com/ntd002/DSC_209_Final_Project/refs/heads/main/static/food-imports.json');
		
		let chart = Sunburst(data, {
			value: d => d.value, // size of each node (file); null for internal nodes (folders)
			label: d => d.name, // display name for each cell
			title: (d, n) => `${n.ancestors().reverse().map(d => d.data.name).join(".")}\n${n.value.toLocaleString("en")}`, // hover text
			width: 1152,
			height: 1152
		});
		
		d3.select(element).append(() => chart);
	});

    function Sunburst(data, { // data is either tabular (array of objects) or hierarchy (nested objects)
		id = Array.isArray(data) ? d => d.id : null, // if tabular data, given a d in data, returns a unique identifier (string)
		parentId = Array.isArray(data) ? d => d.parentId : null, // if tabular data, given a node d, returns its parent’s identifier
		children, // if hierarchical data, given a d in data, returns its children
		value, // given a node d, returns a quantitative value (for area encoding; null for count)
		sort = (a, b) => d3.descending(a.value, b.value), // how to sort nodes prior to layout
		title, // given a node d, returns its hover text
		link, // given a node d, its link (if any)
		linkTarget = "_blank", // the target attribute for links (if any)
		margin = 1, // shorthand for margins
		marginTop = margin, // top margin, in pixels
		marginRight = margin, // right margin, in pixels
		marginBottom = margin, // bottom margin, in pixels
		marginLeft = margin, // left margin, in pixels
		padding = 1, // separation between arcs
		fill = "#ccc", // fill for arcs (if no color encoding)
		fillOpacity = 0.6, // fill opacity for arcs
	} = {}) {

		// If id and parentId options are specified, or the path option, use d3.stratify
		// to convert tabular data to a hierarchy; otherwise we assume that the data is
		// specified as an object {children} with nested objects (a.k.a. the “flare.json”
		// format), and use d3.hierarchy.
		const width = 928;
        const height = width;
        const radius = width / 6;

        // Create the color scale.
        const color = d3.scaleOrdinal(d3.quantize(d3.interpolateRainbow, data.children.length + 1));

        // Compute the layout.
        const hierarchy = d3.hierarchy(data)
            .sum(d => d.value)
            .sort((a, b) => b.value - a.value);
        const root = d3.partition()
            .size([2 * Math.PI, hierarchy.height + 1])
            (hierarchy);
        root.each(d => d.current = d);

        // Create the arc generator.
        const arc = d3.arc()
            .startAngle(d => d.x0)
            .endAngle(d => d.x1)
            .padAngle(d => Math.min((d.x1 - d.x0) / 2, 0.005))
            .padRadius(radius * 1.5)
            .innerRadius(d => d.y0 * radius)
            .outerRadius(d => Math.max(d.y0 * radius, d.y1 * radius - 1))

        // Create the SVG container.
        const svg = d3.create("svg")
            .attr("viewBox", [-width / 2, -height / 2, width, width])
            .style("font", "9px sans-serif");

        // Append the arcs.
        const path = svg.append("g")
            .selectAll("path")
            .data(root.descendants().slice(1))
            .join("path")
            .attr("fill", d => { while (d.depth > 1) d = d.parent; return color(d.data.name); })
            .attr("fill-opacity", d => arcVisible(d.current) ? (d.children ? 0.6 : 0.4) : 0)
            .attr("pointer-events", d => arcVisible(d.current) ? "auto" : "none")

            .attr("d", d => arc(d.current));

        // Make them clickable if they have children.
        path.filter(d => d.children)
            .style("cursor", "pointer")
            .on("click", clicked);

        const format = d3.format(",d");
        path.append("title")
            .text(d => `${d.ancestors().map(d => d.data.name).reverse().join("/")}\n${format(d.value)}`);



        const label = svg.append("g")
            .attr("pointer-events", "none")
            .attr("text-anchor", "middle")
            .style("user-select", "none")
            .selectAll("text")
            .data(root.descendants().slice(1))
            .join("text")
            .attr("dy", "0.35em")
            .attr("fill-opacity", d => +labelVisible(d.current))
            .attr("transform", d => labelTransform(d.current))
            .text(d => d.data.name + 

            (typeof d.data.value==='undefined' ? '' : ": $"+ d.data.value +"m"));

        const parent = svg.append("circle")
            .datum(root)
            .attr("class","circle")
            .attr("r", radius)
            .attr("fill", "none")
            .attr("pointer-events", "all")
            .on("click", clicked);

        // Handle zoom on click.
        function clicked(event, p) {
            parent.datum(p.parent || root);

            root.each(d => d.target = {
            x0: Math.max(0, Math.min(1, (d.x0 - p.x0) / (p.x1 - p.x0))) * 2 * Math.PI,
            x1: Math.max(0, Math.min(1, (d.x1 - p.x0) / (p.x1 - p.x0))) * 2 * Math.PI,
            y0: Math.max(0, d.y0 - p.depth),
            y1: Math.max(0, d.y1 - p.depth)
            });

            const t = svg.transition().duration(750);

            // Transition the data on all arcs, even the ones that aren’t visible,
            // so that if this transition is interrupted, entering arcs will start
            // the next transition from the desired position.
            path.transition(t)
                .tween("data", d => {
                const i = d3.interpolate(d.current, d.target);
                return t => d.current = i(t);
                })
            .filter(function(d) {
                return +this.getAttribute("fill-opacity") || arcVisible(d.target);
            })
                .attr("fill-opacity", d => arcVisible(d.target) ? (d.children ? 0.6 : 0.4) : 0)
                .attr("pointer-events", d => arcVisible(d.target) ? "auto" : "none") 

                .attrTween("d", d => () => arc(d.current));

            label.filter(function(d) {
                return +this.getAttribute("fill-opacity") || labelVisible(d.target);
            }).transition(t)
                .attr("fill-opacity", d => +labelVisible(d.target))
                .attrTween("transform", d => () => labelTransform(d.current));

            changeImage(p["data"]["name"]);
        }
        
        function arcVisible(d) {
            return d.y1 <= 3 && d.y0 >= 1 && d.x1 > d.x0;
        }

        function labelVisible(d) {
            return d.y1 <= 3 && d.y0 >= 1 && (d.y1 - d.y0) * (d.x1 - d.x0) > 0.03;
        }

        function labelTransform(d) {
            const x = (d.x0 + d.x1) / 2 * 180 / Math.PI;
            const y = (d.y0 + d.y1) / 2 * radius;
            return `rotate(${x - 90}) translate(${y},0) rotate(${x < 180 ? 0 : 180})`;
        }

        function changeImage(category) {
            
            let source = "";
            let prevSource = d3.select("img")
            .attr("src");
            if (category === "Animals" || 
                category === "Bovine" ||
                category === "Swine" ||
                category === "Sheep and Goats" ||
                category === "Poultry" ||
                category === "Eggs") {
                source = "images/animal.jpg";
            }
            else if (category === "Beverages" ||
                    category === "Wine" ||
                    category === "Malt Beer" ||
                    category === "Nonalcoholic Beverages" ||
                    category === "Liquors and Liqueurs"
            ) {
                source = "images/bev.jpg";
            }
            else if (category === "Cocoa" ||
                    category === "Cocoa Beans" ||
                    category === "Cocoa Paste, Butter, and Powder" ||
                    category === "Chocolate"
            ) {
                source = "images/choc.jpg";
            }
            else if (category === "Coffee, Tea, and Spices" ||
                    category === "Coffee Beans, Unroasted" ||
                    category === "Coffee, Roasted and Instant" ||
                    category === "Coffee Extracts and Preparations" ||
                    category === "Tea and Mate" ||
                    category === "Spices"
            ) {
                source = "images/coffee.jpg";
            }
            else if (category === "Dairy" ||
                    category === "Cheese, Fresh or Processed" ||
                    category === "Yogurt, Buttermilk, or Whey" ||
                    category === "Milk and Cream" ||
                    category === "Butter or Spreads"
            ) {
                source = "images/dairy.jpg";
            }
            else if (category === "Fish" ||
                    category === "Whole Fish" ||
                    category === "Fish Fillets and Mince" ||
                    category === "Whole Shellfish" ||
                    category === "Prepared Fish and Shellfish"
            ) {
                source = "images/fish.jpg";
            }
            else if (category === "Fruit" ||
            category === "Fresh or Chilled Fruit" ||
            category === "Bananas and Plantains" ||
            category === "Frozen Fruit" ||
            category === "Prepared or Preserved fruit" ||
            category === "Fruit Juices"
            ) {
                source = "images/fruit.png";
            }
            else if (category === "Grains" ||
                    category === "Bulk Grains" ||
                    category === "Wheat and Products" ||
                    category === "Rice and Products" ||
                    category === "Milled Grain Products" ||
                    category === "Cereal and Bakery foods"
            ) {
                source = "images/grain.jpg";
            }
            else if (category === "Meat" || 
                    category === "Fresh or Chilled Red Meat" ||
                    category === "Frozen Red Meat and Parts" ||
                    category === "Fowl and Other Meats" ||
                    category === "Prepared Meats"
            ) {
                source = "images/meat.jpg";
            }
            else if (category === "Nuts" ||
                    category === "Tree Nuts" ||
                    category === "Cashew Nuts" ||
                    category === "Prepared Tree Nuts" ||
                    category === "Ground Nuts"
            ) {
                source = "images/nuts.jpg";
            }
            else if (category === "Sweets" ||
                    category === "Sugar, Cane and Beet" ||
                    category === "Other Sweeteners and Syrups" ||
                    category === "Confections"
            ) {
                source = "images/sweet.jpg";
            }
            else if (category === "Vegetables" ||
            category === "Fresh Vegetables" ||
            category === "Frozen Vegetables" ||
            category === "Dried Vegetables" ||
            category === "Prepared or Preserved Vegetables"
            ) {
                source = "images/veg.jpg";
            }
            else if (category ==="Vegetable Oil and Oilseeds" ||
                    category === "Crude Vegetable Oils" ||
                    category === "Refined Vegetable Oils" ||
                    category === "Olive Oil" ||
                    category === "Tropical Oils" ||
                    category === "Oilseeds"
            ) {
                source = "images/oil.png";
            }
            else if (category ==="Other" ||
                    category === "Sauces, Soups, or Prepared Foods" ||
                    category === "Essential Oils" ||
                    category === "Animal and Other Fats" ||
                    category === "Other Edible Products"
            ) {
                source = "images/other.png";
            }


            //if selecting within the same category, do nothing
            if (category === "Imports") {
                d3.select("img")
                .style('opacity', 1)
                .style('transform', 'scale(1)')
                .transition()
                .duration(750)
                .style('opacity', 0)
                .style('transform', 'scale(0)');
            }
            else if (source !== prevSource) {
                d3.select("img")
                .attr("src",source)
                .style('opacity', 0)
                .style('transform', 'scale(0)')
                .transition()
                .duration(750)
                .style('opacity', 1)
                .style('transform', 'scale(1)');
            }


            
        }

        return svg.node();
    }


</script>

<div bind:this={element}>	
    <img>
</div>

<style>

    img {
        position: absolute;
        pointer-events: none;
        border-radius: 50%;
        width: 25%;
        left: 37.5%;
        top: 89%;
    }
    
</style>