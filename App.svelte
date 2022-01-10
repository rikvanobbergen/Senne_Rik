<!-- The idea of this visualization is to easily make basic data concerning characters, traits and items available
with an easy to use database where people can select either a champion, a trait or an item
and the text and visualisations underneath will give all relevant data, such as cost of characters etc.
However, the data in these basic datasets are hardly interesting to make nice visualisations such as charts:
therefore the idea is to add additional data after submitting a request at riotgames (which Rik has already done).
If possible we could perhaps make a map that would map out how often the selected character is being played where in Flanders.
Or we could make a graph showing the evolution of popularity.
Too many ideas, but up until now too few data... -->
<script>
  //imports
  import * as d3 from "d3";
  import {
    forceSimulation,
    forceLink,
    forceManyBody,
    forceCenter,
    forceCollide
  } from "d3-force";
  import { scaleLinear, scaleBand, scaleTime } from "d3-scale";
  import {
    Graphic,
    Section,
    Line,
    Point,
    RectangleLayer,
    XAxis,
    YAxis,
    nrow
  } from "@snlab/florence";
  import DataContainer from "@snlab/florence-datacontainer";
  import { data_champions, abilities_data } from "./Data_champions.js";
  import {
    data_traits,
    data_origins,
    data_classes,
    data_sets,
    data_style_origins
  } from "./Data_Traits.js";
  import toLowerCase from "/index.js";
  import { data_network, onlynodes, onlylinks } from "./networkdata.js";
  import { classlinks, originlinks } from "./separatedata.js";
  import { legenddata, legenddata2 } from "./legenddata.js";
  import {
    nodecolours,
    nodecolours2,
    linkcolours,
    linkcolours2,
    alllinkcolours
  } from "./networkcolours.js";
  const reload = "javascript:location.reload(true)";
  //Datasets as constants
  const Champions = new DataContainer(data_champions).pivotLonger({
    columns: [
      "Abomination",
      "Assassin",
      "Brawler",
      "Cannoneer",
      "Caretaker",
      "Cavalier",
      "Cruel",
      "Dawnbringer",
      "Draconic",
      "Forgotten",
      "Hellion",
      "Inanimate",
      "Invoker",
      "Ironclad",
      "Knight",
      "Legionnaire",
      "Mystic",
      "Nightbringer",
      "Ranger",
      "Redeemed",
      "Renewer",
      "Revenant",
      "Sentinel",
      "Skirmisher",
      "Spellweaver",
      "Victorious"
    ],
    namesTo: "trait_name",
    valuesTo: "trait_present"
  });
  const Traits = new DataContainer(data_traits);
  const origins = new DataContainer(data_origins);
  const classes = new DataContainer(data_classes);
  const summary = new DataContainer(data_champions).groupBy("name");
  const styles = new DataContainer(data_style_origins);
  const abilities = new DataContainer(abilities_data);

  var Champions_data = "";
  var selected_style_origin;
  var selected_style_champion;
  $: Champions_data = Champions.filter(row => row.name === selected_champion);
  $: Champions_origins = Champions_data.filter(row => row.trait_present === 1);
  $: Champion_origin1 = Champions_origins.domain("trait_name");
  $: Champions_classes = Champions_data.filter(row => row.trait_present === 2);
  $: Traits_champions = Champions.filter(
    row => row.trait_name === selected_origin
  ).filter(row => row.trait_present === 1);
  $: Origins_champions = Champions.filter(
    row => row.trait_name === selected_origin
  ).filter(row => row.trait_present === 1 || row.trait_present === 2);
  $: Classes_champions = Champions.filter(
    row => row.trait_name === selected_class
  ).filter(row => row.trait_present === 1 || row.trait_present === 2);
  $: Origins_data = Traits.filter(row => row.name === selected_origin);
  $: Classes_data = Traits.filter(row => row.name === selected_class);
  $: selected_ability = abilities.filter(row => row.name === selected_champion);
  $: selected_ability_title = selected_ability.domain("title")[0];
  $: selected_ability_description = selected_ability.domain("description")[0];
  //style based on selection of the backgroud of the two first rectangles
  $: {
    selected_style_origin = styles.filter(row => row.trait === selected_origin);
    style_selected_origin = selected_style_origin.domain("style");
    if (selected_champion == "") {
      selected_style_champion = styles.filter(row => row.trait === "");
      style_selected_champion = "#B892B9";
      selected_ability_title = "";
      selected_ability_description = "";
    } else {
      selected_style_champion = styles.filter(
        row => row.trait === Champion_origin1[0]
      );
      style_selected_champion = selected_style_champion.domain("style");
    }
  }
  // functions
  const padding = { left: 80, bottom: 40, top: 5, right: 5 };
  const colour = "rgb(252, 28, 3)";

  //Variables for the reactive statements
  var value = "";
  let selected_champion = "";
  var currentselected = "";
  let selected_origin = "";
  let selected_class = "";
  let Ytraits = 40;
  var Xtrait1 = 24;
  var Xtrait2 = 45;
  var Xtrait3 = 55;
  var text_trait1 = "";
  var text_trait2 = "";
  var trait1 = "";
  var trait2 = "";
  var trait3 = "";
  var last_trait = "";
  var number_traits = "";
  var champion_text = "";
  var cost_text = "";
  var origin_text = "";
  var class1_text = "";
  var class2_text = "";
  var image_champions = "";
  var image_trait1 = "";
  var image_trait2 = "";
  var image_trait3 = "";
  var num_classes = 0;
  var num_origins = 0;
  var style_selected_origin;
  var style_selected_champion;
  // linear scales to automatically scale certain elements
  let Xchampions = scaleLinear()
    .domain([0, 100])
    .range([0, 1000]);
  let Ychampions = scaleLinear()
    .domain([0, 100])
    .range([0, 500]);
  let Xorigins = scaleLinear()
    .domain([0, 100])
    .range([0, 1000]);
  let Yorigins = scaleLinear()
    .domain([0, 100])
    .range([0, 750]);
  //rendering the badge icons in the right place for the second svg
  function Xiconinbadge(input) {
    return Xorigins(input + 1.26);
  }
  function Yiconinbadge(input) {
    return Yorigins(input + 2.2);
  }
  function Xclass(input) {
    return Xorigins(input + 1.26);
  }
  function Yclass(input) {
    return Yorigins(input + 45);
  }
  function Xclass_champion(input) {
    return Xorigins(20 + input * 10);
  }
  // set variables for the text in the origins and classes section
  var X_origins = Xorigins(20);
  //variables for interaction
  var interaction_index1 = "";
  var interaction_index2 = "";
  var interaction_index3 = "";
  var interaction_index4 = "";
  var input = "";
  // functions for interactions
  function trait_icon_dosomething1(input) {
    interaction_index1 = input;
  }
  function trait_icon_dosomething2(input) {
    interaction_index2 = input;
  }
  function trait_icon_dosomething3(input) {
    interaction_index3 = input;
  }
  function champion_icon_dosomething1(input) {
    interaction_index4 = input;
  }
  //loops for data manipulation, if loop for the top part
  // these dependencies are grouped togheter to prevent the cyclical dependency error
  $: {
    currentselected = Champions_origins.domain("trait_name")[0];
    //first the text and image components
    // different if scenarios for no selection, a champion with either one class/ two classes and one/two origins
    if (currentselected === undefined) {
      champion_text = "-- Select a champion--";
      cost_text = "0";
      image_champions = "TrainingDummy";
      text_trait1 = "Origin:";
      text_trait2 = "Class:";
      trait1 = "A training dummy is worthless, select a useful champion to WIN";
      trait2 = "";
      trait3 = "";
      image_trait1 = "empty";
      image_trait2 = "empty";
      image_trait3 = "empty";
      num_origins = 0;
      num_classes = 0;
    } else {
      champion_text = Champions_data.domain("name");
      cost_text = Champions_data.column("cost")[0];
      origin_text = Champions_origins.domain("trait_name");
      class1_text = Champions_classes.domain("trait_name");
      image_champions = champion_text;
      if (origin_text[1] === undefined && class1_text[1] === undefined) {
        Xtrait1 = 24;
        Xtrait2 = 50;
        Xtrait3 = 65;
        trait1 = origin_text[0];
        trait2 = class1_text[0];
        trait3 = "";
        num_origins = 1;
        num_classes = 1;
        text_trait1 = "Origin:";
        text_trait2 = "Class:";
        image_trait1 = tolower(trait1);
        image_trait2 = tolower(trait2);
        image_trait3 = "empty";
      } else if (origin_text[1] === undefined && class1_text[1] != undefined) {
        Xtrait1 = 24;
        Xtrait2 = 50;
        Xtrait3 = 65;
        text_trait1 = "Origin:";
        text_trait2 = "Classes:";
        trait1 = origin_text[0];
        trait2 = class1_text[0];
        trait3 = class1_text[1];
        num_origins = 1;
        num_classes = 2;
        image_trait1 = tolower(trait1);
        image_trait2 = tolower(trait2);
        image_trait3 = tolower(trait3);
      } else if (origin_text[1] != undefined && class1_text[1] === undefined) {
        Xtrait1 = 24;
        Xtrait2 = 38;
        Xtrait3 = 50;
        text_trait1 = "Origins:";
        text_trait2 = "Class:";
        trait1 = origin_text[0];
        trait2 = origin_text[1];
        trait3 = class1_text[0];
        num_origins = 2;
        num_classes = 1;
        image_trait1 = tolower(trait1);
        image_trait2 = tolower(trait2);
        image_trait3 = tolower(trait3);
      }
    }
    //The interaction component as dependency to activate the reactive statement
    //every champion has an origin as their first trait
    if (interaction_index1 == trait1) {
      selected_origin = trait1;
      interaction_index1 = "";
    }
    // champions with just two traits (origin is the step above, now just their class is left)
    if (num_origins === 1 && num_classes === 1 && interaction_index2 == trait2) {
      selected_class = trait2;
      interaction_index2 = "";
    } else if (interaction_index2 != "" && num_classes == 2) {
      selected_class = trait2;
      interaction_index2 = "";
    } else if (interaction_index2 != "" && num_origins == 2) {
      selected_origin = trait2;
      interaction_index2 = "";
      interaction_index3 = "";
    } else if (interaction_index3 != "") {
      selected_class = trait3;
      interaction_index3 = "";
    }
  }
  // interaction on champion icons instead of the trait icons
  $: {
    if (interaction_index4 != "") {
      selected_champion = interaction_index4;
    }
  }
  //functions for transformations of data that gave unexpected results
  function image_champion(input) {
    if (
      input != "Vel'Koz" &&
      input != "Miss Fortune" &&
      input != "Kha'Zix" &&
      input != "Lee Sin"
    ) {
      value = input;
    } else if (
      input != "Miss Fortune" &&
      input != "Kha'Zix" &&
      input != "Lee Sin"
    ) {
      value = "Velkoz";
    } else if (
      input != "Miss Fortune" &&
      input != "Vel'Koz" &&
      input != "Lee Sin"
    ) {
      value = "Khazix";
    } else if (input == "Miss Fortune" && input != "Lee Sin") {
      value = "MissFortune";
    } else if (input != "Miss Fortune" && input == "Lee Sin") {
      value = "LeeSin";
    }
    return "./images_champions/TFT5_" + value + ".png";
  }
  // images are in lower case while champion names and traits contain capital letters
  function tolower(input) {
    return input.toLowerCase();
  }
  function image_trait(input) {
    if (input == "empty") {
      ("./images_traits/empty.png");
    } else {
      return "./images_traits/" + input + ".png";
    }
  }
  function image_origin(input) {
    if (input == "") {
      ("./images_traits/empty.png");
    } else {
      input = tolower(input);
      return "./images_traits/" + input + ".svg";
    }
  }

  /// initiate needed constants
  const width = 1250;
  const height = 700;
  const scaleX = scaleLinear().domain([0, width]);
  const scaleY = scaleLinear().domain([0, height]);
  // functions
  function drag({ dragType, localCoordinates }, index) {
    if (dragType === "start") {
    }
    if (dragType === "drag") {
      nodes[index].x = localCoordinates.x;
      nodes[index].y = localCoordinates.y;
      links[index].x = localCoordinates.x;
      links[index].y = localCoordinates.y;
    }
    if (dragType === "end") {
    }
  }
  /// make data selectable

  var selected_network = "";
  var selected_legend = "";
  var nodes = [...data_network.nodes];
  var links = [...data_network.links];
  var ncolours = nodecolours;
  var lcolours = alllinkcolours;
  var legendlabels = legenddata;
  let draw = true;
  $: if (selected_network === "Only class connections") {
    draw = false;
    nodes = [...classlinks.nodes];
    links = [...classlinks.links];
    simulation = recalcSimulation;
    ncolours = nodecolours;
    lcolours = linkcolours;
    legendlabels = legenddata;
    draw = true;
    selected_legend = "Origins";
  } else if (selected_network === "Only origin connections") {
    draw = false;
    nodes = [...originlinks.nodes];
    links = [...originlinks.links];
    simulation = recalcSimulation;
    ncolours = nodecolours2;
    lcolours = linkcolours2;
    legendlabels = legenddata2;
    draw = true;
    selected_legend = "Classes";
  } else if (selected_network === "") {
    draw = false;
    nodes = [...data_network.nodes];
    links = [...data_network.links];
    simulation = recalcSimulation;
    ncolours = nodecolours;
    lcolours = alllinkcolours;
    legendlabels = legenddata;
    draw = true;
    selected_legend = "Origins";
  } else {
    draw = false;
    nodes = [...data_network.nodes];
    links = [...data_network.links];
    simulation = recalcSimulation;
    ncolours = nodecolours;
    draw = true;
    selected_legend = "Origins";
    lcolours = alllinkcolours;
    legendlabels = legenddata;
  }

  function recalcSimulation() {
    let simulation = forceSimulation(nodes)
      .force("link", forceLink(links).id(d => d.id))
      .force("center", forceManyBody())
      .force("center", forceCenter(width / 2, height / 2))
      .force("charge", forceCollide(25))
      .on("tick", () => {
        nodes = nodes;
        links = links;
      });
    return simulation;
  }
  /// running the force simulation
  var simulation = forceSimulation(nodes)
    .force("link", forceLink(links).id(d => d.id))
    .force("center", forceManyBody())
    .force("center", forceCenter(width / 2 + 100, height / 2))
    .force("charge", forceCollide(35))
    .on("tick", () => {
      nodes = nodes;
      links = links;
    });
  /// change label on:mousover
  let nodeinmaking = {};
  $: label = nodeinmaking.id;
  let linkinmaking = {};
  $: label2 = linkinmaking.value;

  /// allow user to drag the nodes to wherever they like

  let selectedlegend;
</script>
<!-- Here starts the actual visualization -->
<main>
<!-- First some information to introduce users to the dataset -->
  <div class="backgrounddrop" align='center'>
    <h1>League of Legends: Teamfight tactics</h1>
    <h2>Welcome to our interactive website on set Teamfight tactics - set 5.5 - Dawn of heroes!</h2>
    <h3>This game is all about building the best team of champions to go for victory! 
    Thus, knowledge on the base data of the game is essential for doing a good job. 
    Explore the champions of Runeterra by selecting a champion to know more about. </h3>
    <!-- the user can select a champion from the first dropdown menu -->
          <label for="champion_select">Choose a champion:</label>
          <select bind:value={selected_champion} name="name" id="championId">     
            <option value="">-Please Select A Champion-</option>
            {#each Champions.domain('name') as item}
              <option value={item}>{item}</option>
            {/each}
          </select>
          <a href={reload}>Refresh page</a>
        <div align="center">
        <!-- Here follows the visualization that alternates with different selected champions -->
      <svg class="champion" padding="2">
        <!-- rectangles around different sections to make the appearance neat -->
        <g>
          <rect class="border_champion" fill={style_selected_champion}/>
          <rect class="cost_border"     x={Xchampions(23)} y={Ychampions(19)}/>
        </g>
        <!-- All the text data and image data in a seperate g element -->
        <g>
          <text x={Xchampions(2)} y={Ychampions(11)} class="title_name_champion">Champion:{champion_text}</text>
          <text x={Xchampions(24)} y={Ychampions(24.5)} class="title_properties">Cost:</text>
          <text x={Xchampions(32)} y={Ychampions(24.5)} class="cost">{cost_text}</text>
          <text x={Xchampions(23)} y={Ychampions(32)} class="title_properties">Traits:</text>
          <text x={Xchampions(23.5)} y={Ychampions(36)} class="title_properties">{text_trait1}</text>
          <text x={Xchampions(50)} y={Ychampions(36)} class="title_properties">{text_trait2}</text>
          <text x={Xchampions(Xtrait1)} y={Ychampions(Ytraits)} class="text_description1" on:click={() => trait_icon_dosomething1(trait1)}>{trait1}</text>
          <text x={Xchampions(Xtrait2)} y={Ychampions(Ytraits)} class="text_description1" on:click={() => trait_icon_dosomething2(trait2)}>{trait2}</text>
          <text x={Xchampions(Xtrait3)} y={Ychampions(Ytraits)} class="text_description1" on:click={() => trait_icon_dosomething3(trait3)}>{trait3}</text>
          <text x={Xchampions(2)} y={Ychampions(65)} class="title_properties">Ability:{selected_ability_title}</text>
          <switch>
          <!-- foreignobject allows text wrapping in an svg which is otherwise impossible with a regular text element -->
            <foreignObject x={Xchampions(2)} y={Ychampions(62)}  width={800} height={200}>
                  <p class="text_description3" xmlns="http://www.w3.org/1999/xhtml">
                  {selected_ability_description}
                  </p>
            </foreignObject>
          </switch>
        </g>
        <!-- images in a seperate g element -->
        <g>
          <image class="image_champion" x={Xchampions(2)} y={Ychampions(19)} href={image_champion(image_champions)}/>
          <image class="image_trait" x={Xchampions(Xtrait1)} y={Ychampions(Ytraits-22)} href={image_trait(image_trait1)} on:click={() => trait_icon_dosomething1(trait1)}  />
          <image class="image_trait" x={Xchampions(Xtrait2)} y={Ychampions(Ytraits-22)} href={image_trait(image_trait2)} on:click={() => trait_icon_dosomething2(trait2)} />
          <image class="image_trait" x={Xchampions(Xtrait3)} y={Ychampions(Ytraits-22)} href={image_trait(image_trait3)} on:click={() => trait_icon_dosomething3(trait3)} />
        </g>
      </svg>
</div>
<!-- new div for the classes and origins section -->
<!-- these folow the same logic as the champions svg elements -->
  <h3>Every champion has at least one class and one origin (some even have two!). 
  When placing multiple champions with the same trait(s) on your board, your team gains some bonusses.
  Explore the section below to find out which champions work well togheter and what bonusses they povide!
  Keep in mind though, a balanced team with multiple trait bonusses is the key to victory!  </h3>
      <!-- SELECTBOXES ORIGINS AND CLASSES -->
      <div align="center">
          <label for="champion_select">Choose an origin|| </label>
          <label for="champion_select"> --|| Choose a class</label>
        </div> 
        <div align="center">
          <select bind:value={selected_origin} name="name" id="originid">     
              <option value="">--Please Select An Origin--</option>
                {#each origins.domain('name') as item}
                    <option value={item}>{item}</option>
                {/each} -->
          </select>
                    
          <select bind:value={selected_class} name="name" id="id">
            <option value="">--Please Select A Class--</option>
            {#each classes.domain('name') as item}
              <option value={item}>{item}</option>
            {/each} -->
          </select>
        </div>
        <!-- ELEMENTS OF ORIGINS AND CLASSES -->
        <div align="left">
          <svg class="origin">  
          <!-- Rectangles around the different elements -->
            <g>
              <rect x={Xorigins(10)} y={Yorigins(5)} class="border_origin" fill={style_selected_origin}/>
              <rect x={Xorigins(10)} y={Yorigins(50)} class="border_class"/>
            </g>
            <!-- ORIGINS -->
            <g>
              <text x={X_origins} y={Yorigins(10)} class="title_properties">Origin: {Traits_champions.domain('trait_name')}</text>
              <text x={X_origins} y={Yorigins(15)} class="title_properties">Description: </text>
              <switch>
                <foreignObject x={X_origins} y={Yorigins(12.5)} width={700} height={200} >
                  <p class="text_description3" xmlns="http://www.w3.org/1999/xhtml">
                  {Origins_data.domain('description')}
                  </p>
                </foreignObject>
              </switch>
              <text x={X_origins} y={Yorigins(30)} class="title_properties">Champions with this origin:</text>
              {#each Traits_champions.domain('name') as item, i}
                <text class="text_description2" x={Xclass_champion(i)} y={Yorigins(43)} on:click={() => champion_icon_dosomething1(item)}>{item}</text>
                <image class="champion_class" x={Xclass_champion(i)} y={Yorigins(31)} href={image_champion(item)} on:click={() => champion_icon_dosomething1(item)} />
              {/each}          
            </g>
            <!-- CLASSES -->
            <g>
              <text x={X_origins} y={Yclass(10)} class="title_properties">Class: {Classes_data.domain('name')}</text>
              <text x={X_origins} y={Yclass(15)} class="title_properties">Description: </text>
              <switch>
                <foreignObject x={X_origins} y={Yclass(13)}  width={700} height={200}>
                  <p class="text_description3" xmlns="http://www.w3.org/1999/xhtml">
                  {Classes_data.domain('description')}
                  </p>
                </foreignObject>
              </switch>
              <text x={X_origins} y={Yclass(30)} class="title_properties">Champions with this class:</text>
              {#each Classes_champions.domain('name') as item, i}
                <text class="text_description2" x={Xclass_champion(i)} y={Yclass(43)} on:click={() => champion_icon_dosomething1(item)}>{item}</text>
                <image class="champion_class" x={Xclass_champion(i)} y={Yclass(31)} href={image_champion(item)} on:click={() => champion_icon_dosomething1(item)}/>
              {/each}
            </g>
            <!-- BADGES AND TRAIT ICONS -->
            <!-- badges and icons are placed in the corners of the rectangle elements to make them more excited when they explore the dataset -->
            <!-- ORIGINS -->
            <g>
              <image class="badge" x={Xorigins(10)} y={Yorigins(5.5)} href="./images_traits/bg_gold.png"/>
              <image class="image_origin" x={Xiconinbadge(10)} y={Yiconinbadge(5)} href={image_origin(selected_origin)}/>
              <image class="badge" x={Xorigins(10)} y={Yorigins(35)} href="./images_traits/bg_silver.png"/>
              <image class="image_origin" x={Xiconinbadge(10)} y={Yiconinbadge(34.5)} href={image_origin(selected_origin)}/>
              <image class="badge" x={Xorigins(92.5)} y={Yorigins(5.5)} href="./images_traits/bg_bronze.png"/>
              <image class="image_origin" x={Xiconinbadge(92.5)} y={Yiconinbadge(5)} href={image_origin(selected_origin)}/>
            </g>
            <!-- CLASSES -->
            <g>
              <image class="badge" x={Xorigins(10)} y={Yclass(5.5)} href="./images_traits/bg_gold.png"/>
              <image class="image_origin" x={Xiconinbadge(10)} y={Yclass(7.5)} href={image_origin(selected_class)}/>
              <image class="badge" x={Xorigins(10)} y={Yclass(35)} href="./images_traits/bg_silver.png"/>
              <image class="image_origin" x={Xiconinbadge(10)} y={Yclass(37)} href={image_origin(selected_class)}/>
              <image class="badge" x={Xorigins(92.5)} y={Yclass(5.5)} href="./images_traits/bg_bronze.png"/>
              <image class="image_origin" x={Xiconinbadge(92.5)} y={Yclass(7.5)} href={image_origin(selected_class)}/>
            </g>
            
          </svg>
          <!-- DATA NETWORK -->
          <!-- whitespaces to format  -->
      <div>
          {#each Array(40) as i}
            <br/>
          {/each}
      </div>
      <div>
          <label for="network_select">What would you like to see?</label>
          <select bind:value={selected_network} name="network" id="select">
            <option value="">All connections</option>
            <option value="Only class connections">Only class connections</option>
            <option value="Only origin connections">Only origin connections</option>
          </select>
          <br/>
          <p class="text_description3">You can drag the nodes to make things clearer and hover over nodes and links. To find out what you're looking at, see the labels below. </p>
          <p class="title_properties">Node: {label}</p>
          <p class="title_properties">Link: {label2}</p>
          <Graphic {width} {height} {scaleX} {scaleY}>
            {#each links as link}
              <Line
                x={[link.source.x, link.target.x]}
                y={[link.source.y, link.target.y]}
                stroke={lcolours[link.value]}
                strokeWidth={1}
                onMouseover={l => linkinmaking = link}
              />
            {/each}
            {#each nodes as node, i}
              <Point
                x={node.x}
                y={node.y}
                radius={10}
                fill={ncolours[node.group]}
                onMouseover={n => nodeinmaking = node}
                onMousedrag={n => drag(n,i)}
                
              />
            {/each}
            <!-- Adding legend -->
            <rect class="legend"></rect>
            {#each legendlabels as legend, i}
              <text class="title_properties" x={15} y={30}>LEGEND: {selected_legend}</text>
              <rect stroke={"#252525"} fill={legend.colour} width={40} height={20} x={15} y={50+i*30}/>
              <text class="text_description3" x={70} y={66+i*30}>{legend.id}</text>
            {/each}
          </Graphic>
        </div>
    </div>
  </div>
</main>
<!-- STYLE SECTION -->
<style>
  /* TEXT style */
  main {
    font-family: sans-serif;
    text-align: center;
  }
  h1,
  label {
    font-family: "Press Start 2P", cursive;
  }
  h2,
  h3 {
    font-family: "Teko", cursive;
    font-size: 20px;
  }
  .text_description1 {
    font-family: "Teko", cursive;
    font-size: 17px;
  }
  .text_description1:hover {
    filter: drop-shadow(4px 4px 4px rgb(255 255 51));
  }
  .text_description2 {
    font-family: "Teko", cursive;
    font-size: 15px;
  }
  .text_description2:hover {
    filter: drop-shadow(4px 4px 4px rgb(255 255 51));
  }
  .text_description3 {
    font-family: "Teko", cursive;
    font-size: 17px;
  }
  .title_name_champion {
    font-size: 25px;
    font-family: "Press Start 2P", cursive;
  }
  .title_properties,
  .cost {
    font-family: "Press Start 2P", cursive;
  }

  /* Image and boxes style */
  .image_champion {
    width: 200;
    height: 200;
    border-radius: 20;
    box-shadow: 0px 0px 300px #fff;
    filter: drop-shadow(3px 5px 2px rgb(0 0 0 / 0.4));
  }
  .badge {
    width: 70;
    height: 70;
    filter: drop-shadow(3px 5px 2px rgb(0 0 0 / 0.4));
  }
  .image_trait {
    width: 50;
    height: 300;
    border-radius: 20;
    alt: "./images_traits/empty.png";
    filter: drop-shadow(3px 5px 2px rgb(0 0 0 / 0.4));
  }
  .image_trait {
    width: 50;
    height: 300;
    border-radius: 20;
    alt: "./images_traits/empty.png";
    filter: drop-shadow(3px 5px 2px rgb(0 0 0 / 0.4));
  }
  .image_trait:hover {
    filter: drop-shadow(6px 10px 4px rgb(51 255 153 / 0.4));
  }
  .image_origin {
    width: 45;
    height: 45;
    filter: drop-shadow(3px 5px 2px rgb(0 0 0 / 0.4));
  }
  .champion_class {
    width: 70;
    height: 70;
    filter: drop-shadow(3px 5px 2px rgb(0 0 0 / 0.4));
  }
  .champion_class:hover {
    filter: drop-shadow(6px 10px 4px rgb(51 255 153 / 0.4));
  }
  .border_champion {
    width: 900px;
    height: 450px;
    fill-opacity: 50%;
    stroke-width: 4px;
    stroke: black;
    filter: drop-shadow(3px 5px 2px rgb(0 0 0 / 0.4));
  }
  .border_origin {
    width: 900px;
    height: 300px;
    fill-opacity: 50%;
    stroke-width: 4px;
    stroke: black;
    filter: drop-shadow(3px 5px 2px rgb(0 0 0 / 0.4));
  }
  .border_class {
    width: 900px;
    height: 300px;
    fill: #b892b9;
    fill-opacity: 50%;
    stroke-width: 4px;
    stroke: black;
    filter: drop-shadow(3px 5px 2px rgb(0 0 0 / 0.4));
  }
  .cost_border {
    width: 200px;
    height: 40px;
    fill: gold;
    fill-opacity: 60%;
    stroke-width: 2px;
    stroke: black;
    filter: drop-shadow(3px 5px 2px rgb(0 0 0 / 0.4));
  }

  /* SVG ELEMENTS */
  .champion {
    width: 1000px;
    height: 500px;
    border: 5px black;
  }

  .origin {
    position: absolute;
    width: 1000px;
    height: 700px;
    border: 5px black;
  }
  .backgrounddrop {
    position: absolute;
    width: 1250px;
    height: 2500px;
    background-image: linear-gradient(to bottom right, purple, beige);
  }
  .legend {
    width: 300px;
    height: 440px;
    fill: "none";
    fill-opacity: 0%;
    stroke-width: 4px;
    stroke: black;
    fill: #b892b9;
    fill-opacity: 50%;
  }
</style>