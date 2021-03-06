---
id: litvis

narrative-schemas:
  - ../narrative-schemas/courseworkPG.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

@import "../css/datavis.less"

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

### INM402 Postgraduate Coursework:
# Using networks to visualize the structure of the Chilean economy
**Author: Joaquin Perez-Lapillo**

This project report summarises the design choices made for answering research questions regarding the structure of the Chilean economy using visualization techniques.

The datasets were obtained from the Central Bank of Chile, where I work as an economic analyst. All data is available on the [Statistics Division webpage](https://www.bcentral.cl/web/guest/cuentas-nacionales-anuales).
The Input-Output tables containing transactions between industries are published yearly in two aggregation levels: 12 and 111 industries. Given this, I decided to create different designs depending on the level of aggregation, for the last available datasets (2015).

This project and its results will be useful for increasing the visibility and impact of the underlying datasets, considering that these statistics are currently published as Excel tables. Additionally, they will be useful for both academia and the training of new professionals in the National Accounts area.

It is also relevant to mention that in order to obtain clean and tidy datasets, I pre-processed the data on a [Jupyter notebook](preprocessing.ipynb), which is included in this folder.

{(questions|}

-	How Chilean industries interact with each other?
-	Which industries are more relevant in terms of connectivity? Which ones are more independent?
- Are the physical goods industries more dependent on each other than the service-based industries?

{|questions)}

{(visualization|}
### 1. Network graph of the Chilean economy structure (screenshot from [Gephi](/industriesGroup.gephi))

<img alt="preview1.png" src="assets/preview1.png" width="500" height="500" >

### 2. Interactive version (screenshots from [Gephi](/industriesGroup.gephi))
#### 2.1. Interactive view: default look
<img alt="screenshot_155613.png" src="assets/screenshot_155613.png" width="70%"  >

#### 2.2. Examples: selecting Manufacturing (left) and Business services (right)
<img alt="screenshot_155714.png" src="assets/screenshot_155714.png" width="49%"  > <img alt="screenshot_155725.png" src="assets/screenshot_155725.png" width="49%"  >

### 3. Matrix representation: detailed level (111 industries)

```elm {v interactive}
matrix : Spec
matrix =
    let
        edgeData =
            dataFromColumns []
                << dataColumn "industry1" (strColumn "industry1" edgeDetailTable |> strs)
                << dataColumn "industry2" (strColumn "industry2" edgeDetailTable |> strs)
                << dataColumn "value" (numColumn "value" edgeDetailTable |> nums)

        nodeData =
            dataFromColumns []
                << dataColumn "industry" (strColumn "industry" nodeDetailTable |> strs)
                << dataColumn "order"
                    (List.range 1 (List.length (strColumn "industry" nodeDetailTable))
                        |> List.map toFloat
                        |> nums
                    )
                << dataColumn "VA" (numColumn "VA" nodeDetailTable |> nums)

        trans =
            transform
                << lookup "industry1" (nodeData []) "industry" [ "order" ]
                << calculateAs "datum.order" "colOrder"
                << lookup "industry2" (nodeData []) "industry" [ "order" ]
                << calculateAs "datum.order" "rowOrder"

        encMatrixBL =
            encoding
                << position X [ pName "colOrder", pMType Ordinal, pAxis [] ]
                << position Y [ pName "rowOrder", pMType Ordinal, pAxis [] ]
                << size [ mName "value", mMType Quantitative ]
                << tooltips
                    [ [ tName "industry1", tMType Nominal ]
                    , [ tName "industry2", tMType Nominal ]
                    ]

        specMatrixBL =
            asSpec [ edgeData [], trans [], encMatrixBL [], circle [ maColor "purple"] ]

        encMatrixTR =
            encoding
                << position Y [ pName "colOrder", pMType Ordinal, pAxis [] ]
                << position X [ pName "rowOrder", pMType Ordinal, pAxis [] ]
                << size [ mName "value", mMType Quantitative ]
                << tooltips
                    [ [ tName "industry1", tMType Nominal ]
                    , [ tName "industry2", tMType Nominal ]
                    ]

        specMatrixTR =
            asSpec [ edgeData [], trans [], encMatrixTR [], circle [maColor "purple"] ]

        encRowLabels =
            encoding
                << position X [ pNum 0 ]
                << position Y [ pName "order", pMType Ordinal, pAxis [] ]
                << text [ tName "industry" ]

        specRowLabels =
            asSpec [ nodeData [], encRowLabels [], textMark [ maAlign haRight, maSize 6.5, maDx -5 ] ]

        encColLabels =
            encoding
                << position X [ pName "order", pMType Ordinal, pAxis [] ]
                << position Y [ pNum 0 ]
                << text [ tName "industry" ]

        specColLabels =
            asSpec [ nodeData [], encColLabels [], textMark [ maAlign haRight, maAngle 90, maSize 6.5 ] ]
    in
    toVegaLite [ width 700, height 700, layer [ specMatrixBL, specMatrixTR, specRowLabels, specColLabels ] ]
```
{|visualization)}

{(insights|}

Both designs for the 12-industries aggregation level (the still image and the interactive version) allow obtaining insights for answering the research questions of this project, while the matrix at high detail level provides granular information to strengthen the answers provided.

Regarding the first research question "How Chilean industries interact with each other?" we can say that all industries are connected between each other, but some of them can be further classified as "backwards-chained" and others as "forward-chained" industries according to their relationship with others. For example, consider Business services, a highly forward-chained industry dedicated to providing essential services to all others, and Personal services, an industry which only receives products and services from others (backwards-chained). From the first graph, we can also deduce that some industries also provide relevant inputs to themselves, a phenomenon called "self-consumption". That is the case of Manufacturing, Construction, and Transportation.

Now, with respect to the degree of dependence of each element of the system, Manufacturing is clearly the most connected industry, being strongly chained both backwards and forwards to a variety of relevant industries such as Agriculture, forestry and fishing (backwards) and Construction (forward). In other words, Manufacturing acts like an "attractive force" to the other industries, keeping the network together. From the matrix design, we can see that the relationship between Agriculture and Manufacturing is mostly explained by the supply of species such as agricultural products, cattle, chickens, pigs, and fish to the food industry, and forestry products for the wood industry.
On the other hand, examples of relatively "independent" industries are Governmental services and Banking and finance, which have only weak connections to all other industries.

The connectivity between "physical goods" industries seems to be intensive, especially what happens in between Agriculture-Manufacturing-Construction. It is interesting to notice that Mining is not highly connected to them, although it is known to be a key industry in the Chilean economy. However, from the detailed matrix, we can see that Mining is only connected in a relevant way to the supply of electric power and engineering services.
The group of industries dedicated to providing services are also dependant on each other. An example of this is the connection between Business services-Wholesale and retail trade-Transportation.

{|insights)}

{(designJustification|}

My first approach was to use VegaLite to build the network graphs for both the aggregate and detail level. This attempt is shown in a separated [markdown document](/PreviousDesignVegaLite.m). As seen in lectures and in this example, VegaLite seems to be inefficient to display networks representations to the level required in this project, since it does not allow the use of algorithms for a better layout of the networks. That was the main reason why I chose to build the designs using Gephi, an open-source, specialized software for visualizing network graphs. This decision involved investing time to learn the software and to adapt the datasets to the format required for using Gephi (edges and nodes tables should have specific structures).

The layout algorithm I used to create the designs for the aggregate level (still image and interactive view) is [Fruchterman Reingold](#references), a force-directed layout method. The decision behind this selection of algorithm type is based on [Muntzner](#references), who mention force-directed methods as one of the most common approaches for problems with not too large datasets. In more detail, the algorithm considers the strength of the connection between nodes to group them together, with the intention of minimize the _energy_ of the system. This method showed to be much well-suited for this problem than other available options such as the Yifan Hu and Force Atlas algorithms. The result is a balanced network, circle-shaped, placing the most dense-connected nodes in the centre. This creates a very intuitive sense of _gravitational pull_ from these more connected nodes to the less integrated ones, which are then represented as _satellites_ of the system. Given all this "physics" context, I decided to create first the interactive design (using the _Overview_ option in Gephi) including a dark background, which evokes space, with each industry representing a planet. In the case of the still image (which was created using the _Preview_ option in Gephi), I created a design inspired on the visualization of [flight routes on the globe](https://thekidshouldseethis.com/post/flight-visualizations-over-24-hour-periods). In this case, we can consider the flights as the flow of goods and services from one industry to another, and the most connected industries as hub airports.

Regarding the colour decision for representing the nodes, I decided to use a palette of soft colours available in Gephi, given that industries are discrete categories that have no intrinsic order (_Nominal_).

Designing how to visualize the edges involved some experimentation. Especially, finding the thickness ratio that represented the best how intensive was the flow of goods and services between industries. Another decision made for the edges was to exclude their labels, mainly for following [Tufte's](#references) principle of maximizing the data-ink ratio. In other words, having a minimalistic design should be easier to read, focusing on the "ink" areas that lead to obtaining relevant insights for answering the research questions. Of course, the actual weights associated with each edge are always visible when using the interactive tool (in [Gephi](/industriesGroup.gephi)).

The case of the detailed dataset (111-industries level) was challenging since I had much more nodes to represent on the design than with the previous dataset (12-industries). A first approach on [Gephi](/industriesDetail.gephi) is available on a separated [markdown document](/PreviousDesignGephi.m), showing both a still image and an interactive tool for visualizing the detailed dataset. However, and as [Robert Kosara notes on his webpage](https://eagereyes.org/techniques/graphs-hairball), and also supported by [Muntzner](#references), a network with more than a dozen of nodes-edges starts to resemble as a hairball. This was particularly true for this design: it was simply impossible to read for knowledge extraction. The solution was to use a mirror matrix representation on VegaLite, as suggested in [lectures](#references), visualizing how intensive the connection between industries is according to the size of the circle. Lastly, the choice of colour (purple) for the circles was due to consistency with the aggregate level designs.

{|designJustification)}

{(validation|}

With respect to how well the designs are for answering the research questions, I am confident that both the still image and the interactive view are very useful, powerful tools to show the relationship between industries in a simple and intuitive way. Nevertheless, if I was to do this project again, I would probably choose the colours for the nodes more carefully, trying palettes that can allow a better distinction between elements.
The level of detail of these designs is not much though, and that is when the matrix representation can be used. However, extract knowledge from a matrix representation of 111x111 elements is not an easy task. It could require more context information for the readers, e.g. a brief introduction of what is shown in the image. Also, there is always the conflict of showing just one of the diagonals and leave an empty space or showing the complete mirror representation (which I think can be more useful for fast searches). The interactivity in the matrix case was key since the font size of the labels cannot be increased too much in order to have a clean design. In summary, I would say that in this case there is a clear trade-off between adding more detailed information (nodes and edges) and how useful the visual representation is.

This project showed the importance of evaluating the designs by constantly asking for opinions to people with different backgrounds. Economists and National Accounts specialists are used to reading input-output matrices, but that is not the case for engineers or business analysts, for example.

For future work, I think an interesting design can be created using an intermediate level of aggregation (probably using 24-industries). This could allow easy visualization of interesting relationships at a more detailed level. In addition, I would like to apply this approach to other statistics such as international trade between countries and money flows of the financial system.

In terms of technology, Gephi proved to be very efficient for creating a visual representation of networks, but not very flexible for some aspects such as the position of the nodes' labels or the shape of the arrows. I would still recommend it for future work representations with not very large datasets.

{|validation)}

{(references|}

**Fruchterman, T. M. J., & Reingold, E. M.** (1991) Graph Drawing by Force-Directed Placement. Software: Practice and Experience, 21(11).
**Munzner, T.** (2015) Chapter 9: _Arrange Networks and Trees_, pp.200-217 in Visualization Analysis and Design, CRC Press.
**Tufte, E.** (2001) The Visual Display of Quantitative Information, Graphics Press.
**Wood, J.** (2019) Lecture notes, Data Visualization module, City, University of London

{|references)}


### Appendix
Data:
1. Node table: aggregation level of 12 industries
```elm {l=hidden}
nodeGroupTable : Table
nodeGroupTable =
  """industryGroup,VA,P,rx,ry
Agriculture forestry and fishing,5805.960386,13287.97591,751,594
Banking and finance,7495.246072,12613.87587,235,633
Business services,16561.30109,24331.72012,396,864
Construction,10498.09541,23814.12036,570,882
Governmental services,7497.93612,10736.60407,280,978
Manufacturing,18605.82162,54431.84294,162,260
Mining,13685.7428,25085.8297,77,205
Personal services,17674.48195,25376.76217,677,470
Real estate and housing,11922.08051,15591.36619,22,360
Transportation and communication,13197.49913,28808.83484,536,683
Utilities,4719.020235,11809.47839,599,11
Wholesale and retail trade food and accommodation,18021.17375,35909.21532,159,620"""
  |> fromCSV
```
2. Edge table: aggregation level of 12 industries
```elm {l=hidden}
edgeGroupTable : Table
edgeGroupTable =
  """industry1,industry2,value
Agriculture forestry and fishing,Agriculture forestry and fishing,1835.268762
Agriculture forestry and fishing,Mining,1.288602644
Agriculture forestry and fishing,Manufacturing,7072.184816
Agriculture forestry and fishing,Utilities,36.55020548
Agriculture forestry and fishing,Construction,5.628000462
Agriculture forestry and fishing,Wholesale and retail trade food and accommodation,293.5160256
Agriculture forestry and fishing,Transportation and communication,4.279968365
Agriculture forestry and fishing,Banking and finance,3.538886921
Agriculture forestry and fishing,Real estate and housing,0.683068057
Agriculture forestry and fishing,Business services,18.00176002
Agriculture forestry and fishing,Personal services,41.28041904
Agriculture forestry and fishing,Governmental services,39.91955689
Mining,Agriculture forestry and fishing,102.0930458
Mining,Mining,1505.464082
Mining,Manufacturing,1302.660074
Mining,Utilities,49.72179843
Mining,Construction,99.22163167
Mining,Wholesale and retail trade food and accommodation,58.73411085
Mining,Transportation and communication,34.72677188
Mining,Banking and finance,26.95376791
Mining,Real estate and housing,6.486451653
Mining,Business services,54.90076239
Mining,Personal services,23.77124417
Mining,Governmental services,8.101982888
Manufacturing,Agriculture forestry and fishing,2434.333545
Manufacturing,Mining,1470.005552
Manufacturing,Manufacturing,6673.906416
Manufacturing,Utilities,512.9864376
Manufacturing,Construction,4339.472072
Manufacturing,Wholesale and retail trade food and accommodation,2505.838966
Manufacturing,Transportation and communication,1478.675179
Manufacturing,Banking and finance,175.7408445
Manufacturing,Real estate and housing,39.15475759
Manufacturing,Business services,576.9892825
Manufacturing,Personal services,1112.040736
Manufacturing,Governmental services,257.7568391
Utilities,Agriculture forestry and fishing,102.8816941
Utilities,Mining,1782.764833
Utilities,Manufacturing,1624.885869
Utilities,Utilities,3624.329638
Utilities,Construction,108.3454937
Utilities,Wholesale and retail trade food and accommodation,530.7577691
Utilities,Transportation and communication,278.8855459
Utilities,Banking and finance,63.47418642
Utilities,Real estate and housing,94.87405354
Utilities,Business services,167.1341489
Utilities,Personal services,393.6007216
Utilities,Governmental services,399.1617232
Construction,Agriculture forestry and fishing,27.14588221
Construction,Mining,16.75507325
Construction,Manufacturing,53.35290506
Construction,Utilities,97.53993417
Construction,Construction,2845.121777
Construction,Wholesale and retail trade food and accommodation,280.9218347
Construction,Transportation and communication,164.6896925
Construction,Banking and finance,18.94129401
Construction,Real estate and housing,1986.064733
Construction,Business services,84.63590541
Construction,Personal services,307.2679064
Construction,Governmental services,280.1767788
Wholesale and retail trade food and accommodation,Agriculture forestry and fishing,614.5417137
Wholesale and retail trade food and accommodation,Mining,743.3706715
Wholesale and retail trade food and accommodation,Manufacturing,1981.092021
Wholesale and retail trade food and accommodation,Utilities,242.679076
Wholesale and retail trade food and accommodation,Construction,1279.710292
Wholesale and retail trade food and accommodation,Wholesale and retail trade food and accommodation,2554.682504
Wholesale and retail trade food and accommodation,Transportation and communication,1769.879418
Wholesale and retail trade food and accommodation,Banking and finance,169.3773014
Wholesale and retail trade food and accommodation,Real estate and housing,57.59139259
Wholesale and retail trade food and accommodation,Business services,738.4212009
Wholesale and retail trade food and accommodation,Personal services,1068.918083
Wholesale and retail trade food and accommodation,Governmental services,209.4901377
Transportation and communication,Agriculture forestry and fishing,483.4159708
Transportation and communication,Mining,989.9349564
Transportation and communication,Manufacturing,2854.458171
Transportation and communication,Utilities,309.9189537
Transportation and communication,Construction,475.065674
Transportation and communication,Wholesale and retail trade food and accommodation,3386.856335
Transportation and communication,Transportation and communication,4104.719353
Transportation and communication,Banking and finance,522.6389649
Transportation and communication,Real estate and housing,56.86001127
Transportation and communication,Business services,1078.096121
Transportation and communication,Personal services,480.6943689
Transportation and communication,Governmental services,381.7358576
Banking and finance,Agriculture forestry and fishing,399.3647931
Banking and finance,Mining,203.1863581
Banking and finance,Manufacturing,840.0637438
Banking and finance,Utilities,224.0615487
Banking and finance,Construction,616.8138341
Banking and finance,Wholesale and retail trade food and accommodation,1201.461514
Banking and finance,Transportation and communication,560.3411183
Banking and finance,Banking and finance,1411.797541
Banking and finance,Real estate and housing,758.7284197
Banking and finance,Business services,446.3500573
Banking and finance,Personal services,221.8280396
Banking and finance,Governmental services,18.60912042
Real estate and housing,Agriculture forestry and fishing,49.17578476
Real estate and housing,Mining,73.75826179
Real estate and housing,Manufacturing,277.931743
Real estate and housing,Utilities,35.09845218
Real estate and housing,Construction,77.18826503
Real estate and housing,Wholesale and retail trade food and accommodation,1858.744796
Real estate and housing,Transportation and communication,584.6196063
Real estate and housing,Banking and finance,141.8729785
Real estate and housing,Real estate and housing,254.320969
Real estate and housing,Business services,603.4263701
Real estate and housing,Personal services,622.190385
Real estate and housing,Governmental services,91.60419792
Business services,Agriculture forestry and fishing,373.9098375
Business services,Mining,2656.614834
Business services,Manufacturing,3153.926822
Business services,Utilities,492.3772313
Business services,Construction,1288.37261
Business services,Wholesale and retail trade food and accommodation,3211.736081
Business services,Transportation and communication,2224.974832
Business services,Banking and finance,1222.950387
Business services,Real estate and housing,305.002953
Business services,Business services,2882.698826
Business services,Personal services,1066.401378
Business services,Governmental services,502.0781419
Personal services,Agriculture forestry and fishing,9.838461286
Personal services,Mining,34.73306398
Personal services,Manufacturing,151.9365221
Personal services,Utilities,14.54462468
Personal services,Construction,23.25678085
Personal services,Wholesale and retail trade food and accommodation,132.4128811
Personal services,Transportation and communication,150.1145563
Personal services,Banking and finance,44.22096141
Personal services,Real estate and housing,11.06256392
Personal services,Business services,82.58987154
Personal services,Personal services,904.2820322
Personal services,Governmental services,36.36434026
Governmental services,Agriculture forestry and fishing,12.61175157
Governmental services,Mining,40.86117582
Governmental services,Manufacturing,120.0696988
Governmental services,Utilities,19.06435369
Governmental services,Construction,17.63458364
Governmental services,Wholesale and retail trade food and accommodation,146.2867738
Governmental services,Transportation and communication,104.7977127
Governmental services,Banking and finance,11.76963868
Governmental services,Real estate and housing,4.426700699
Governmental services,Business services,27.19296154
Governmental services,Personal services,42.81781822
Governmental services,Governmental services,36.90386245"""
  |> fromCSV
```
3. Node table: aggregation level of 111 industries
```elm {l=hidden}
nodeDetailTable : Table
nodeDetailTable =
  """industry,VA,P,industryGroup,rx,ry
Cultivation of annual crops,409.7164535,1342.197694,Agriculture forestry and fishing,604,217
Cultivation of vegetables,591.8868372,881.4436027,Agriculture forestry and fishing,893,747
Cultivation of grapes,483.2474956,895.8856528,Agriculture forestry and fishing,432,761
Cultivation of other fruits,1514.826372,2356.979722,Agriculture forestry and fishing,675,818
Cattle bredding,388.523852,952.8834383,Agriculture forestry and fishing,407,832
Pigs breeding,198.4449663,615.223293,Agriculture forestry and fishing,34,953
Chicken breeding,289.3844038,1059.143955,Agriculture forestry and fishing,736,288
Breeding of other animals,66.53200832,127.1660045,Agriculture forestry and fishing,272,564
Farming services,328.7859374,544.6959779,Agriculture forestry and fishing,866,864
Forestry,930.8244835,1872.614611,Agriculture forestry and fishing,402,25
Aquaculture,152.3109672,2030.909458,Agriculture forestry and fishing,13,810
Extractive fishing,451.4766088,608.8325048,Agriculture forestry and fishing,151,366
Coal mining,26.75517313,53.64606013,Mining,282,332
Crude oil and gas mining,30.39660754,139.7589213,Mining,753,537
Copper mining,12435.52492,22456.70989,Mining,66,550
Iron mining,134.9434714,526.4717103,Mining,122,201
Mining of other metals,362.9910272,757.3955301,Mining,648,659
Other mining activities and mining services,695.1315931,1151.847592,Mining,407,124
Processing and preserving of meat,702.092009,3303.738615,Manufacturing,345,320
Processing and preserving of fishmeal and fish oil,313.9683064,719.825054,Manufacturing,398,677
Processing and preserving of fish,757.5952711,3351.065052,Manufacturing,607,554
Processing and preserving of fruits and vegetables,418.5118215,1316.905007,Manufacturing,322,697
Manufacture of vegetable and animal oils,102.5437837,339.3117214,Manufacturing,967,653
Manufacture of dairy products,732.7771175,1899.618532,Manufacturing,784,125
Manufacture of grain mill products,160.0885944,753.1021634,Manufacturing,784,476
Manufacture of prepared animal feeds,424.9336289,2230.059909,Manufacturing,692,834
Manufacture of bakery products,810.7022717,1619.487676,Manufacturing,57,395
Manufacture of macaroni noodles and other related products,86.77926618,172.2254666,Manufacturing,587,292
Manufacture of other food products,589.0687494,1426.762739,Manufacturing,956,616
Distilling rectifying and blending of spirits,42.26172242,177.9170099,Manufacturing,753,577
Manufacture of wines,804.0131919,1825.960913,Manufacturing,806,347
Manufacture of beers,139.0622077,545.5780864,Manufacturing,691,581
Manufacture of soft drinks,730.4862404,1755.449538,Manufacturing,898,104
Manufacture of tobacco products,1006.650603,1156.972833,Manufacturing,602,244
Manufacture of textile products,121.7871434,438.4763916,Manufacturing,62,719
Manufacture of wearing apparel,167.3405103,632.4130487,Manufacturing,314,593
Manufacture of leather products,12.35558819,64.80295374,Manufacturing,459,798
Manufacture of footwear,40.70302688,143.291528,Manufacturing,605,22
Sawmilling and planing of wood,533.1709686,1549.391865,Manufacturing,413,308
Manufacture of wood products,412.5301974,1224.654228,Manufacturing,976,777
Manufacture of pulp and paper,1159.195423,2986.513482,Manufacturing,906,155
Manufacture of containers of paper,185.2911065,715.4687662,Manufacturing,381,888
Manufacture of other paper products,125.8898001,600.9070349,Manufacturing,331,516
Printing,215.0601895,707.3464139,Manufacturing,84,132
Manufacture of refined petroleum products,1728.693277,4468.03521,Manufacturing,751,745
Manufacture of basic chemicals,825.1345108,2712.630766,Manufacturing,718,907
Manufacture of paints,90.86201796,301.7959887,Manufacturing,190,181
Manufacture of pharmaceutical products,414.5298182,1176.806698,Manufacturing,617,872
Manufacture of soap detergents and toilet products,147.6053138,615.4878505,Manufacturing,820,961
Manufacture of other chemical products,198.7117876,666.2371819,Manufacturing,916,529
Manufacture of rubber products,145.5458397,401.3960494,Manufacturing,451,737
Manufacture of plastic products,505.1768598,1803.702173,Manufacturing,553,992
Manufacture of glass and glass products,137.4526799,365.4488104,Manufacturing,839,410
Manufacture of cement,116.7373344,567.7282196,Manufacturing,484,481
Manufacture of concrete and concrete products,322.5142468,1340.289739,Manufacturing,831,945
Manufacture of basic iron and steel,157.6093392,998.5469985,Manufacturing,680,135
Manufacture of other basic metals,105.4598504,493.0386709,Manufacturing,867,166
Manufacture of fabricated metal products,1080.560423,2420.639922,Manufacturing,825,262
Manufacture of industrial and domestic machinery and equipment,560.651197,1355.72139,Manufacturing,547,838
Manufacture of electric and electronic machinery and equipment,351.544055,847.5426872,Manufacturing,171,622
Manufacture of transport equipment,206.8772329,581.6921654,Manufacturing,65,41
Manufacture of furniture,199.2244411,719.1565862,Manufacturing,653,248
Repair and installation of machinery and other manufacturing,516.0726513,938.6998048,Manufacturing,265,893
Electric power generation,2367.054642,4859.963494,Utilities,616,223
Transmission of electric power,370.5737448,431.3356692,Utilities,941,886
Distribution of electric power,606.0914399,3452.327649,Utilities,31,477
Gas and steam manufacture and supply,335.7707212,1315.594335,Utilities,654,825
Water collection treatment and supply,696.1964023,1105.978476,Utilities,534,674
Waste collection and recycling activities,343.3332842,644.278767,Utilities,640,884
Construction of residential buildings,2302.220285,5671.55084,Construction,571,381
Construction of non-residential buildings,1286.127553,3607.337962,Construction,953,511
Civil engineering,3480.371187,7977.188438,Construction,862,537
Specialized construction activities,3429.376383,6558.043117,Construction,450,561
Wholesale and retail trade of motor vehicles,1723.508776,3354.92481,Wholesale and retail trade food and accommodation,69,644
Wholesale trade,6946.195081,13988.69169,Wholesale and retail trade food and accommodation,155,49
Retail trade,6132.302805,12009.56124,Wholesale and retail trade food and accommodation,210,478
Accommodation,626.6221306,1220.602843,Wholesale and retail trade food and accommodation,855,248
Food and beverage service activities,2592.544959,5335.434739,Wholesale and retail trade food and accommodation,436,855
Transport via railways,73.92126583,221.2134026,Transportation and communication,979,617
Other passenger land transport,2312.129547,4665.856006,Transportation and communication,802,964
Freight transport by road,1747.062458,5447.066589,Transportation and communication,373,91
Transport via pipeline,114.617638,154.0766722,Transportation and communication,175,440
Water transport,255.8098772,873.0886763,Transportation and communication,511,528
Air transport,1390.168544,3181.030816,Transportation and communication,559,752
Warehousing,415.7586404,630.2125827,Transportation and communication,736,975
Support activities for land transport,761.5781762,1182.047777,Transportation and communication,931,626
Other support activities for transport,1337.607704,2133.76627,Transportation and communication,627,179
Postal and courier activities,177.3140824,302.6886537,Transportation and communication,420,463
Wireless telecommunications,1163.536537,2685.85608,Transportation and communication,473,916
Wired telecommunications,617.6187024,1359.83638,Transportation and communication,299,894
Other telecommunications activities,507.6498703,1759.812485,Transportation and communication,329,925
Information services activities,1860.113018,2974.054875,Transportation and communication,483,655
Publishing activities,462.6130679,1238.227571,Transportation and communication,827,616
Banking monetary intermediation,5809.74054,8190.92003,Banking and finance,3,293
Insurance activities,709.3250243,2371.120435,Banking and finance,724,525
Auxiliary financial activities,976.1805081,2051.835411,Banking and finance,175,553
Real estate activities,3688.810047,5207.414889,Real estate and housing,959,134
Housing services,8233.270464,10383.9513,Real estate and housing,236,633
Legal and accounting services,2015.392902,2583.803105,Business services,210,8
Architectural and engineering services,4240.891457,6340.062076,Business services,530,890
Other professional activities,4192.283607,6121.569972,Business services,558,270
Rental and leasing activities,1563.454707,2628.397304,Business services,939,368
Business support activities,4549.27842,6657.88766,Business services,359,958
Public administration,7497.93612,10736.60407,Governmental services,289,142
Public education,4745.496271,5884.843278,Personal services,991,887
Private education,2898.583065,3943.727713,Personal services,887,28
Public health activities,3368.332804,4959.985811,Personal services,498,13
Private health activities,3541.152925,6085.426731,Personal services,682,61
Activities of organizations,716.86323,1205.854296,Personal services,456,141
Artistic and entertainment activities,723.3684071,1303.13509,Personal services,188,487
Other personal services,1680.685249,1993.789255,Personal services,252,715"""
  |> fromCSV
```
4. Edge table: aggregation level of 111 industries
```elm {l=hidden}
edgeDetailTable : Table
edgeDetailTable =
  """industry1,industry2,value
Cultivation of annual crops,Cultivation of annual crops,54478.71714
Cultivation of annual crops,Cultivation of vegetables,31276.46783
Cultivation of annual crops,Cultivation of grapes,638.8900463
Cultivation of annual crops,Cultivation of other fruits,6066.879496
Cultivation of annual crops,Cattle bredding,158210.9957
Cultivation of annual crops,Pigs breeding,856.4402206
Cultivation of annual crops,Chicken breeding,41302.2808
Cultivation of annual crops,Breeding of other animals,11324.87387
Cultivation of annual crops,Farming services,735.700495
Cultivation of annual crops,Forestry,23901.40681
Cultivation of annual crops,Aquaculture,10.74775058
Cultivation of annual crops,Extractive fishing,134.233378
Cultivation of annual crops,Coal mining,0.316755605
Cultivation of annual crops,Crude oil and gas mining,0.824473797
Cultivation of annual crops,Copper mining,581.5455676
Cultivation of annual crops,Iron mining,0.478742811
Cultivation of annual crops,Mining of other metals,1.676547585
Cultivation of annual crops,Other mining activities and mining services,5.369532768
Cultivation of annual crops,Processing and preserving of meat,701.8307146
Cultivation of annual crops,Processing and preserving of fishmeal and fish oil,16.70885368
Cultivation of annual crops,Processing and preserving of fish,252.0573486
Cultivation of annual crops,Processing and preserving of fruits and vegetables,18625.63668
Cultivation of annual crops,Manufacture of vegetable and animal oils,34958.5559
Cultivation of annual crops,Manufacture of dairy products,238.1981415
Cultivation of annual crops,Manufacture of grain mill products,206262.6977
Cultivation of annual crops,Manufacture of prepared animal feeds,168987.2145
Cultivation of annual crops,Manufacture of bakery products,24867.98811
Cultivation of annual crops,Manufacture of macaroni noodles and other related products,11.42775445
Cultivation of annual crops,Manufacture of other food products,61308.46006
Cultivation of annual crops,Distilling rectifying and blending of spirits,52.45993341
Cultivation of annual crops,Manufacture of wines,426.4893349
Cultivation of annual crops,Manufacture of beers,2116.167269
Cultivation of annual crops,Manufacture of soft drinks,319.9970379
Cultivation of annual crops,Manufacture of tobacco products,5496.489524
Cultivation of annual crops,Manufacture of textile products,2200.424443
Cultivation of annual crops,Manufacture of wearing apparel,154.1340878
Cultivation of annual crops,Manufacture of leather products,4.142671479
Cultivation of annual crops,Manufacture of footwear,13.22970688
Cultivation of annual crops,Sawmilling and planing of wood,225.11256
Cultivation of annual crops,Manufacture of wood products,207.3198973
Cultivation of annual crops,Manufacture of pulp and paper,200.6524668
Cultivation of annual crops,Manufacture of containers of paper,75.31417982
Cultivation of annual crops,Manufacture of other paper products,149.951203
Cultivation of annual crops,Printing,109.0193772
Cultivation of annual crops,Manufacture of refined petroleum products,147.653122
Cultivation of annual crops,Manufacture of basic chemicals,160.3201164
Cultivation of annual crops,Manufacture of paints,56.8571099
Cultivation of annual crops,Manufacture of pharmaceutical products,857.8053714
Cultivation of annual crops,Manufacture of soap detergents and toilet products,517.7181551
Cultivation of annual crops,Manufacture of other chemical products,106.2636819
Cultivation of annual crops,Manufacture of rubber products,2.258750203
Cultivation of annual crops,Manufacture of plastic products,105.5041981
Cultivation of annual crops,Manufacture of glass and glass products,25.20603523
Cultivation of annual crops,Manufacture of cement,94.51039146
Cultivation of annual crops,Manufacture of concrete and concrete products,263.1085629
Cultivation of annual crops,Manufacture of basic iron and steel,62.94308307
Cultivation of annual crops,Manufacture of other basic metals,28.39892344
Cultivation of annual crops,Manufacture of fabricated metal products,324.939367
Cultivation of annual crops,Manufacture of industrial and domestic machinery and equipment,152.6085721
Cultivation of annual crops,Manufacture of electric and electronic machinery and equipment,51.07644274
Cultivation of annual crops,Manufacture of transport equipment,94.61056552
Cultivation of annual crops,Manufacture of furniture,87.79519356
Cultivation of annual crops,Repair and installation of machinery and other manufacturing,66.5542059
Cultivation of annual crops,Electric power generation,123.0253645
Cultivation of annual crops,Transmission of electric power,1.423858291
Cultivation of annual crops,Distribution of electric power,13.16285215
Cultivation of annual crops,Gas and steam manufacture and supply,368.3765903
Cultivation of annual crops,Water collection treatment and supply,20.38917896
Cultivation of annual crops,Waste collection and recycling activities,12.92523207
Cultivation of annual crops,Construction of residential buildings,687.758395
Cultivation of annual crops,Construction of non-residential buildings,407.3287351
Cultivation of annual crops,Civil engineering,2444.947164
Cultivation of annual crops,Specialized construction activities,696.116143
Cultivation of annual crops,Wholesale and retail trade of motor vehicles,298.9399028
Cultivation of annual crops,Wholesale trade,10172.48531
Cultivation of annual crops,Retail trade,1232.40027
Cultivation of annual crops,Accommodation,106.3342287
Cultivation of annual crops,Food and beverage service activities,64484.43028
Cultivation of annual crops,Transport via railways,20.91543768
Cultivation of annual crops,Other passenger land transport,45.27627482
Cultivation of annual crops,Freight transport by road,59.24387324
Cultivation of annual crops,Transport via pipeline,0.093749662
Cultivation of annual crops,Water transport,11.41178818
Cultivation of annual crops,Air transport,125.5910376
Cultivation of annual crops,Warehousing,53.18161907
Cultivation of annual crops,Support activities for land transport,89.65900483
Cultivation of annual crops,Other support activities for transport,31.34602033
Cultivation of annual crops,Postal and courier activities,6.859361939
Cultivation of annual crops,Wireless telecommunications,250.8528344
Cultivation of annual crops,Wired telecommunications,80.41391218
Cultivation of annual crops,Other telecommunications activities,317.7820378
Cultivation of annual crops,Information services activities,506.7279712
Cultivation of annual crops,Publishing activities,222.7802344
Cultivation of annual crops,Banking monetary intermediation,879.9727419
Cultivation of annual crops,Insurance activities,520.7456455
Cultivation of annual crops,Auxiliary financial activities,225.5031492
Cultivation of annual crops,Real estate activities,464.1964748
Cultivation of annual crops,Housing services,0
Cultivation of annual crops,Legal and accounting services,336.2421246
Cultivation of annual crops,Architectural and engineering services,664.8014457
Cultivation of annual crops,Other professional activities,4328.027342
Cultivation of annual crops,Rental and leasing activities,217.3693133
Cultivation of annual crops,Business support activities,7145.521209
Cultivation of annual crops,Public administration,5241.174915
Cultivation of annual crops,Public education,383.8252888
Cultivation of annual crops,Private education,622.9609722
Cultivation of annual crops,Public health activities,869.6823375
Cultivation of annual crops,Private health activities,333.2759799
Cultivation of annual crops,Activities of organizations,86.98948567
Cultivation of annual crops,Artistic and entertainment activities,124.913887
Cultivation of annual crops,Other personal services,9.445046281
Cultivation of vegetables,Cultivation of annual crops,11508.27842
Cultivation of vegetables,Cultivation of vegetables,43796.0607
Cultivation of vegetables,Cultivation of grapes,131.8789955
Cultivation of vegetables,Cultivation of other fruits,1279.248586
Cultivation of vegetables,Cattle bredding,33076.51525
Cultivation of vegetables,Pigs breeding,177.461086
Cultivation of vegetables,Chicken breeding,8632.952446
Cultivation of vegetables,Breeding of other animals,2377.342687
Cultivation of vegetables,Farming services,153.6421477
Cultivation of vegetables,Forestry,4997.020459
Cultivation of vegetables,Aquaculture,0.327864387
Cultivation of vegetables,Extractive fishing,28.08456327
Cultivation of vegetables,Coal mining,0.009599594
Cultivation of vegetables,Crude oil and gas mining,0.12119993
Cultivation of vegetables,Copper mining,42.30071266
Cultivation of vegetables,Iron mining,0.023927568
Cultivation of vegetables,Mining of other metals,0.104595007
Cultivation of vegetables,Other mining activities and mining services,0.199467073
Cultivation of vegetables,Processing and preserving of meat,143.4675285
Cultivation of vegetables,Processing and preserving of fishmeal and fish oil,0.562096589
Cultivation of vegetables,Processing and preserving of fish,7.876496121
Cultivation of vegetables,Processing and preserving of fruits and vegetables,54368.56782
Cultivation of vegetables,Manufacture of vegetable and animal oils,7303.998761
Cultivation of vegetables,Manufacture of dairy products,22.32830587
Cultivation of vegetables,Manufacture of grain mill products,43109.81184
Cultivation of vegetables,Manufacture of prepared animal feeds,35308.86368
Cultivation of vegetables,Manufacture of bakery products,5243.164813
Cultivation of vegetables,Manufacture of macaroni noodles and other related products,13.33689484
Cultivation of vegetables,Manufacture of other food products,19264.36206
Cultivation of vegetables,Distilling rectifying and blending of spirits,1.61757482
Cultivation of vegetables,Manufacture of wines,30.7346827
Cultivation of vegetables,Manufacture of beers,413.4177693
Cultivation of vegetables,Manufacture of soft drinks,17.08018734
Cultivation of vegetables,Manufacture of tobacco products,1141.194929
Cultivation of vegetables,Manufacture of textile products,451.7308781
Cultivation of vegetables,Manufacture of wearing apparel,9.285763232
Cultivation of vegetables,Manufacture of leather products,0.127120473
Cultivation of vegetables,Manufacture of footwear,0.429882138
Cultivation of vegetables,Sawmilling and planing of wood,10.59232048
Cultivation of vegetables,Manufacture of wood products,7.918783767
Cultivation of vegetables,Manufacture of pulp and paper,8.991317257
Cultivation of vegetables,Manufacture of containers of paper,2.774807394
Cultivation of vegetables,Manufacture of other paper products,8.05718882
Cultivation of vegetables,Printing,5.373684139
Cultivation of vegetables,Manufacture of refined petroleum products,8.49082618
Cultivation of vegetables,Manufacture of basic chemicals,8.354966883
Cultivation of vegetables,Manufacture of paints,2.779165703
Cultivation of vegetables,Manufacture of pharmaceutical products,105.3727411
Cultivation of vegetables,Manufacture of soap detergents and toilet products,79.68701304
Cultivation of vegetables,Manufacture of other chemical products,53.27924443
Cultivation of vegetables,Manufacture of rubber products,0.221193451
Cultivation of vegetables,Manufacture of plastic products,3.622343735
Cultivation of vegetables,Manufacture of glass and glass products,1.718477367
Cultivation of vegetables,Manufacture of cement,3.73923632
Cultivation of vegetables,Manufacture of concrete and concrete products,10.80623373
Cultivation of vegetables,Manufacture of basic iron and steel,2.039048981
Cultivation of vegetables,Manufacture of other basic metals,1.734401852
Cultivation of vegetables,Manufacture of fabricated metal products,19.34523833
Cultivation of vegetables,Manufacture of industrial and domestic machinery and equipment,5.019817253
Cultivation of vegetables,Manufacture of electric and electronic machinery and equipment,3.198938287
Cultivation of vegetables,Manufacture of transport equipment,3.997230078
Cultivation of vegetables,Manufacture of furniture,8.657157432
Cultivation of vegetables,Repair and installation of machinery and other manufacturing,2.589211205
Cultivation of vegetables,Electric power generation,4.127920712
Cultivation of vegetables,Transmission of electric power,0.198972828
Cultivation of vegetables,Distribution of electric power,3.314008047
Cultivation of vegetables,Gas and steam manufacture and supply,11.56670704
Cultivation of vegetables,Water collection treatment and supply,1.268979985
Cultivation of vegetables,Waste collection and recycling activities,1.369195577
Cultivation of vegetables,Construction of residential buildings,79.54043408
Cultivation of vegetables,Construction of non-residential buildings,62.68836963
Cultivation of vegetables,Civil engineering,511.3283208
Cultivation of vegetables,Specialized construction activities,144.4552847
Cultivation of vegetables,Wholesale and retail trade of motor vehicles,12.27288522
Cultivation of vegetables,Wholesale trade,1814.855291
Cultivation of vegetables,Retail trade,61.55628629
Cultivation of vegetables,Accommodation,10182.2936
Cultivation of vegetables,Food and beverage service activities,57206.14031
Cultivation of vegetables,Transport via railways,0.947806802
Cultivation of vegetables,Other passenger land transport,1.862919372
Cultivation of vegetables,Freight transport by road,2.914484247
Cultivation of vegetables,Transport via pipeline,0.044053042
Cultivation of vegetables,Water transport,0.755127271
Cultivation of vegetables,Air transport,11.78196906
Cultivation of vegetables,Warehousing,1.691400546
Cultivation of vegetables,Support activities for land transport,3.866185372
Cultivation of vegetables,Other support activities for transport,6.620679595
Cultivation of vegetables,Postal and courier activities,0.207879802
Cultivation of vegetables,Wireless telecommunications,11.4346622
Cultivation of vegetables,Wired telecommunications,14.80930794
Cultivation of vegetables,Other telecommunications activities,11.75026686
Cultivation of vegetables,Information services activities,77.70283309
Cultivation of vegetables,Publishing activities,8.605330249
Cultivation of vegetables,Banking monetary intermediation,78.03740843
Cultivation of vegetables,Insurance activities,26.84036872
Cultivation of vegetables,Auxiliary financial activities,17.05653574
Cultivation of vegetables,Real estate activities,15.71412326
Cultivation of vegetables,Housing services,0
Cultivation of vegetables,Legal and accounting services,12.46323907
Cultivation of vegetables,Architectural and engineering services,31.23474484
Cultivation of vegetables,Other professional activities,681.6270073
Cultivation of vegetables,Rental and leasing activities,10.31989342
Cultivation of vegetables,Business support activities,1327.869006
Cultivation of vegetables,Public administration,8271.884068
Cultivation of vegetables,Public education,1094.263489
Cultivation of vegetables,Private education,1232.350023
Cultivation of vegetables,Public health activities,4241.916502
Cultivation of vegetables,Private health activities,5065.399385
Cultivation of vegetables,Activities of organizations,11254.30419
Cultivation of vegetables,Artistic and entertainment activities,6.990679947
Cultivation of vegetables,Other personal services,0.903739006
Cultivation of grapes,Cultivation of annual crops,39.57161075
Cultivation of grapes,Cultivation of vegetables,22.55069287
Cultivation of grapes,Cultivation of grapes,0.790065911
Cultivation of grapes,Cultivation of other fruits,5.033277617
Cultivation of grapes,Cattle bredding,114.3921503
Cultivation of grapes,Pigs breeding,0.683723186
Cultivation of grapes,Chicken breeding,29.89172457
Cultivation of grapes,Breeding of other animals,8.190351911
Cultivation of grapes,Farming services,0.641245616
Cultivation of grapes,Forestry,17.51931645
Cultivation of grapes,Aquaculture,0.056088474
Cultivation of grapes,Extractive fishing,0.149816751
Cultivation of grapes,Coal mining,0.001516437
Cultivation of grapes,Crude oil and gas mining,0.212019119
Cultivation of grapes,Copper mining,56.14948613
Cultivation of grapes,Iron mining,0.022661095
Cultivation of grapes,Mining of other metals,0.124343448
Cultivation of grapes,Other mining activities and mining services,0.105156109
Cultivation of grapes,Processing and preserving of meat,4.239443184
Cultivation of grapes,Processing and preserving of fishmeal and fish oil,0.200488518
Cultivation of grapes,Processing and preserving of fish,1.720637271
Cultivation of grapes,Processing and preserving of fruits and vegetables,12448.73414
Cultivation of grapes,Manufacture of vegetable and animal oils,25.40903798
Cultivation of grapes,Manufacture of dairy products,13.91008007
Cultivation of grapes,Manufacture of grain mill products,154.0675165
Cultivation of grapes,Manufacture of prepared animal feeds,125.2072438
Cultivation of grapes,Manufacture of bakery products,27.07299884
Cultivation of grapes,Manufacture of macaroni noodles and other related products,0.145636028
Cultivation of grapes,Manufacture of other food products,55.687139
Cultivation of grapes,Distilling rectifying and blending of spirits,17825.17903
Cultivation of grapes,Manufacture of wines,233677.195
Cultivation of grapes,Manufacture of beers,4.444963618
Cultivation of grapes,Manufacture of soft drinks,17.49710885
Cultivation of grapes,Manufacture of tobacco products,4.153743961
Cultivation of grapes,Manufacture of textile products,5.147307154
Cultivation of grapes,Manufacture of wearing apparel,10.71742571
Cultivation of grapes,Manufacture of leather products,0.023233735
Cultivation of grapes,Manufacture of footwear,0.125927637
Cultivation of grapes,Sawmilling and planing of wood,9.230870063
Cultivation of grapes,Manufacture of wood products,4.5300063
Cultivation of grapes,Manufacture of pulp and paper,7.254540784
Cultivation of grapes,Manufacture of containers of paper,1.425289767
Cultivation of grapes,Manufacture of other paper products,8.314620808
Cultivation of grapes,Printing,4.997965023
Cultivation of grapes,Manufacture of refined petroleum products,9.39202332
Cultivation of grapes,Manufacture of basic chemicals,8.32865968
Cultivation of grapes,Manufacture of paints,2.556030441
Cultivation of grapes,Manufacture of pharmaceutical products,16.65974548
Cultivation of grapes,Manufacture of soap detergents and toilet products,11.84191306
Cultivation of grapes,Manufacture of other chemical products,1.117836432
Cultivation of grapes,Manufacture of rubber products,0.341129961
Cultivation of grapes,Manufacture of plastic products,1.424056795
Cultivation of grapes,Manufacture of glass and glass products,2.185061985
Cultivation of grapes,Manufacture of cement,2.344752315
Cultivation of grapes,Manufacture of concrete and concrete products,7.38512406
Cultivation of grapes,Manufacture of basic iron and steel,0.585708521
Cultivation of grapes,Manufacture of other basic metals,2.025524095
Cultivation of grapes,Manufacture of fabricated metal products,22.09526914
Cultivation of grapes,Manufacture of industrial and domestic machinery and equipment,1.584531505
Cultivation of grapes,Manufacture of electric and electronic machinery and equipment,3.815025096
Cultivation of grapes,Manufacture of transport equipment,2.896606108
Cultivation of grapes,Manufacture of furniture,13.38825878
Cultivation of grapes,Repair and installation of machinery and other manufacturing,1.55610914
Cultivation of grapes,Electric power generation,1.452965366
Cultivation of grapes,Transmission of electric power,0.343797313
Cultivation of grapes,Distribution of electric power,6.367225964
Cultivation of grapes,Gas and steam manufacture and supply,2.634422028
Cultivation of grapes,Water collection treatment and supply,1.505611527
Cultivation of grapes,Waste collection and recycling activities,2.175793002
Cultivation of grapes,Construction of residential buildings,3.118699687
Cultivation of grapes,Construction of non-residential buildings,1.183386512
Cultivation of grapes,Civil engineering,6.318977087
Cultivation of grapes,Specialized construction activities,1.290141051
Cultivation of grapes,Wholesale and retail trade of motor vehicles,8.380070031
Cultivation of grapes,Wholesale trade,74.89495712
Cultivation of grapes,Retail trade,58.25063504
Cultivation of grapes,Accommodation,337.5075298
Cultivation of grapes,Food and beverage service activities,3629.249363
Cultivation of grapes,Transport via railways,0.779067907
Cultivation of grapes,Other passenger land transport,1.278113929
Cultivation of grapes,Freight transport by road,2.703669419
Cultivation of grapes,Transport via pipeline,0.089573968
Cultivation of grapes,Water transport,0.939749021
Cultivation of grapes,Air transport,17.84980806
Cultivation of grapes,Warehousing,0.426915391
Cultivation of grapes,Support activities for land transport,2.91402793
Cultivation of grapes,Other support activities for transport,12.4135886
Cultivation of grapes,Postal and courier activities,0.032838527
Cultivation of grapes,Wireless telecommunications,9.48873675
Cultivation of grapes,Wired telecommunications,27.14138719
Cultivation of grapes,Other telecommunications activities,6.105141782
Cultivation of grapes,Information services activities,137.2557964
Cultivation of grapes,Publishing activities,5.075481938
Cultivation of grapes,Banking monetary intermediation,115.3037543
Cultivation of grapes,Insurance activities,26.40856132
Cultivation of grapes,Auxiliary financial activities,23.18671102
Cultivation of grapes,Real estate activities,5.782352742
Cultivation of grapes,Housing services,0
Cultivation of grapes,Legal and accounting services,6.525523273
Cultivation of grapes,Architectural and engineering services,27.16011149
Cultivation of grapes,Other professional activities,81.55513877
Cultivation of grapes,Rental and leasing activities,9.112139354
Cultivation of grapes,Business support activities,51.45439114
Cultivation of grapes,Public administration,283.0015107
Cultivation of grapes,Public education,2.202108679
Cultivation of grapes,Private education,5.726779513
Cultivation of grapes,Public health activities,19.63301612
Cultivation of grapes,Private health activities,656.5898857
Cultivation of grapes,Activities of organizations,485.5635753
Cultivation of grapes,Artistic and entertainment activities,7.380606758
Cultivation of grapes,Other personal services,1.380622825
Cultivation of other fruits,Cultivation of annual crops,2477.081355
Cultivation of other fruits,Cultivation of vegetables,1410.379543
Cultivation of other fruits,Cultivation of grapes,34.09607045
Cultivation of other fruits,Cultivation of other fruits,276.9051014
Cultivation of other fruits,Cattle bredding,7193.047105
Cultivation of other fruits,Pigs breeding,38.96070549
Cultivation of other fruits,Chicken breeding,1900.494672
Cultivation of other fruits,Breeding of other animals,549.7204858
Cultivation of other fruits,Farming services,33.67467524
Cultivation of other fruits,Forestry,1087.197867
Cultivation of other fruits,Aquaculture,0.451784467
Cultivation of other fruits,Extractive fishing,6.219610265
Cultivation of other fruits,Coal mining,0.013012112
Cultivation of other fruits,Crude oil and gas mining,0.495113753
Cultivation of other fruits,Copper mining,142.1875633
Cultivation of other fruits,Iron mining,0.064819899
Cultivation of other fruits,Mining of other metals,0.326718186
Cultivation of other fruits,Other mining activities and mining services,0.39669813
Cultivation of other fruits,Processing and preserving of meat,134.9955121
Cultivation of other fruits,Processing and preserving of fishmeal and fish oil,0.953500101
Cultivation of other fruits,Processing and preserving of fish,11.49362406
Cultivation of other fruits,Processing and preserving of fruits and vegetables,378772.1242
Cultivation of other fruits,Manufacture of vegetable and animal oils,1589.332836
Cultivation of other fruits,Manufacture of dairy products,2042.936014
Cultivation of other fruits,Manufacture of grain mill products,9450.604294
Cultivation of other fruits,Manufacture of prepared animal feeds,7688.058606
Cultivation of other fruits,Manufacture of bakery products,7993.75429
Cultivation of other fruits,Manufacture of macaroni noodles and other related products,0.783772652
Cultivation of other fruits,Manufacture of other food products,18754.28069
Cultivation of other fruits,Distilling rectifying and blending of spirits,2.287928989
Cultivation of other fruits,Manufacture of wines,43.24344957
Cultivation of other fruits,Manufacture of beers,100.4224761
Cultivation of other fruits,Manufacture of soft drinks,48.53611887
Cultivation of other fruits,Manufacture of tobacco products,249.7009989
Cultivation of other fruits,Manufacture of textile products,107.1879455
Cultivation of other fruits,Manufacture of wearing apparel,28.45389587
Cultivation of other fruits,Manufacture of leather products,0.177717634
Cultivation of other fruits,Manufacture of footwear,0.682218213
Cultivation of other fruits,Sawmilling and planing of wood,27.32105463
Cultivation of other fruits,Manufacture of wood products,16.35830073
Cultivation of other fruits,Manufacture of pulp and paper,22.19479319
Cultivation of other fruits,Manufacture of containers of paper,5.4541047
Cultivation of other fruits,Manufacture of other paper products,23.00002454
Cultivation of other fruits,Printing,14.40074469
Cultivation of other fruits,Manufacture of refined petroleum products,25.31834769
Cultivation of other fruits,Manufacture of basic chemicals,23.34705332
Cultivation of other fruits,Manufacture of paints,7.398352192
Cultivation of other fruits,Manufacture of pharmaceutical products,68.2156548
Cultivation of other fruits,Manufacture of soap detergents and toilet products,46.44849027
Cultivation of other fruits,Manufacture of other chemical products,5.704895459
Cultivation of other fruits,Manufacture of rubber products,0.825018852
Cultivation of other fruits,Manufacture of plastic products,6.371165991
Cultivation of other fruits,Manufacture of glass and glass products,5.611697405
Cultivation of other fruits,Manufacture of cement,8.077176695
Cultivation of other fruits,Manufacture of concrete and concrete products,24.38710439
Cultivation of other fruits,Manufacture of basic iron and steel,3.216049748
Cultivation of other fruits,Manufacture of other basic metals,5.355318056
Cultivation of other fruits,Manufacture of fabricated metal products,58.87969517
Cultivation of other fruits,Manufacture of industrial and domestic machinery and equipment,8.16202073
Cultivation of other fruits,Manufacture of electric and electronic machinery and equipment,10.01311741
Cultivation of other fruits,Manufacture of transport equipment,9.303553857
Cultivation of other fruits,Manufacture of furniture,32.35334439
Cultivation of other fruits,Repair and installation of machinery and other manufacturing,5.47721059
Cultivation of other fruits,Electric power generation,6.969063826
Cultivation of other fruits,Transmission of electric power,0.8054952
Cultivation of other fruits,Distribution of electric power,14.51561651
Cultivation of other fruits,Gas and steam manufacture and supply,17.06314169
Cultivation of other fruits,Water collection treatment and supply,3.958767172
Cultivation of other fruits,Waste collection and recycling activities,5.216993054
Cultivation of other fruits,Construction of residential buildings,32.22586531
Cultivation of other fruits,Construction of non-residential buildings,18.79282668
Cultivation of other fruits,Civil engineering,121.1049629
Cultivation of other fruits,Specialized construction activities,33.28606881
Cultivation of other fruits,Wholesale and retail trade of motor vehicles,27.68432452
Cultivation of other fruits,Wholesale trade,9934.192757
Cultivation of other fruits,Retail trade,2446.864007
Cultivation of other fruits,Accommodation,7859.04109
Cultivation of other fruits,Food and beverage service activities,73857.2695
Cultivation of other fruits,Transport via railways,2.364230726
Cultivation of other fruits,Other passenger land transport,4.212691711
Cultivation of other fruits,Freight transport by road,7.798348914
Cultivation of other fruits,Transport via pipeline,0.201419881
Cultivation of other fruits,Water transport,2.430875362
Cultivation of other fruits,Air transport,43.39503025
Cultivation of other fruits,Warehousing,2.566643162
Cultivation of other fruits,Support activities for land transport,9.191315371
Cultivation of other fruits,Other support activities for transport,28.47291049
Cultivation of other fruits,Postal and courier activities,0.28177809
Cultivation of other fruits,Wireless telecommunications,28.67690699
Cultivation of other fruits,Wired telecommunications,62.61578919
Cultivation of other fruits,Other telecommunications activities,23.21542414
Cultivation of other fruits,Information services activities,319.7010026
Cultivation of other fruits,Publishing activities,18.03851472
Cultivation of other fruits,Banking monetary intermediation,282.410272
Cultivation of other fruits,Insurance activities,74.40680999
Cultivation of other fruits,Auxiliary financial activities,58.269648
Cultivation of other fruits,Real estate activities,26.96066499
Cultivation of other fruits,Housing services,0
Cultivation of other fruits,Legal and accounting services,24.70972784
Cultivation of other fruits,Architectural and engineering services,80.46173846
Cultivation of other fruits,Other professional activities,351.3825397
Cultivation of other fruits,Rental and leasing activities,26.82194536
Cultivation of other fruits,Business support activities,420.2012842
Cultivation of other fruits,Public administration,4806.148602
Cultivation of other fruits,Public education,375.4048268
Cultivation of other fruits,Private education,584.8465089
Cultivation of other fruits,Public health activities,1684.11088
Cultivation of other fruits,Private health activities,4202.693263
Cultivation of other fruits,Activities of organizations,4097.118039
Cultivation of other fruits,Artistic and entertainment activities,64.07402417
Cultivation of other fruits,Other personal services,3.348264574
Cattle bredding,Cultivation of annual crops,14.15481631
Cattle bredding,Cultivation of vegetables,0.164949601
Cattle bredding,Cultivation of grapes,0.29798134
Cattle bredding,Cultivation of other fruits,0.599394639
Cattle bredding,Cattle bredding,7139.307048
Cattle bredding,Pigs breeding,5501.556442
Cattle bredding,Chicken breeding,40035.31038
Cattle bredding,Breeding of other animals,14.50007075
Cattle bredding,Farming services,0.09881312
Cattle bredding,Forestry,0.251389331
Cattle bredding,Aquaculture,6.83940373
Cattle bredding,Extractive fishing,0.046974967
Cattle bredding,Coal mining,0.001334858
Cattle bredding,Crude oil and gas mining,0.187782123
Cattle bredding,Copper mining,49.72113694
Cattle bredding,Iron mining,0.02006025
Cattle bredding,Mining of other metals,0.110097534
Cattle bredding,Other mining activities and mining services,0.093003882
Cattle bredding,Processing and preserving of meat,486805.9544
Cattle bredding,Processing and preserving of fishmeal and fish oil,15.65560531
Cattle bredding,Processing and preserving of fish,1.517448349
Cattle bredding,Processing and preserving of fruits and vegetables,1.82594157
Cattle bredding,Manufacture of vegetable and animal oils,842.0720035
Cattle bredding,Manufacture of dairy products,237944.6398
Cattle bredding,Manufacture of grain mill products,14.98463831
Cattle bredding,Manufacture of prepared animal feeds,3.039741079
Cattle bredding,Manufacture of bakery products,2642.742263
Cattle bredding,Manufacture of macaroni noodles and other related products,0.123436957
Cattle bredding,Manufacture of other food products,4473.470471
Cattle bredding,Distilling rectifying and blending of spirits,0.274183242
Cattle bredding,Manufacture of wines,1672.854166
Cattle bredding,Manufacture of beers,2.688445116
Cattle bredding,Manufacture of soft drinks,15.49025591
Cattle bredding,Manufacture of tobacco products,0.195584108
Cattle bredding,Manufacture of textile products,4800.049094
Cattle bredding,Manufacture of wearing apparel,9.489285284
Cattle bredding,Manufacture of leather products,563.5778411
Cattle bredding,Manufacture of footwear,0.558534793
Cattle bredding,Sawmilling and planing of wood,8.170633898
Cattle bredding,Manufacture of wood products,4.007137147
Cattle bredding,Manufacture of pulp and paper,6.420672956
Cattle bredding,Manufacture of containers of paper,1.26051094
Cattle bredding,Manufacture of other paper products,7.361020777
Cattle bredding,Printing,4.42425095
Cattle bredding,Manufacture of refined petroleum products,8.315431575
Cattle bredding,Manufacture of basic chemicals,7.373181808
Cattle bredding,Manufacture of paints,2.262595705
Cattle bredding,Manufacture of pharmaceutical products,245.6019651
Cattle bredding,Manufacture of soap detergents and toilet products,14.37092061
Cattle bredding,Manufacture of other chemical products,0.987362607
Cattle bredding,Manufacture of rubber products,0.302108969
Cattle bredding,Manufacture of plastic products,1.25861976
Cattle bredding,Manufacture of glass and glass products,1.934834079
Cattle bredding,Manufacture of cement,2.074451567
Cattle bredding,Manufacture of concrete and concrete products,7.373649697
Cattle bredding,Manufacture of basic iron and steel,0.517147556
Cattle bredding,Manufacture of other basic metals,1.793432861
Cattle bredding,Manufacture of fabricated metal products,19.56311928
Cattle bredding,Manufacture of industrial and domestic machinery and equipment,1.399519601
Cattle bredding,Manufacture of electric and electronic machinery and equipment,3.377950777
Cattle bredding,Manufacture of transport equipment,2.563274261
Cattle bredding,Manufacture of furniture,14.06721153
Cattle bredding,Repair and installation of machinery and other manufacturing,1.376620989
Cattle bredding,Electric power generation,1.283762892
Cattle bredding,Transmission of electric power,0.304493773
Cattle bredding,Distribution of electric power,5.639662556
Cattle bredding,Gas and steam manufacture and supply,2.323789136
Cattle bredding,Water collection treatment and supply,1.333112655
Cattle bredding,Waste collection and recycling activities,1.926948759
Cattle bredding,Construction of residential buildings,2.545072026
Cattle bredding,Construction of non-residential buildings,0.865602583
Cattle bredding,Civil engineering,4.042090165
Cattle bredding,Specialized construction activities,0.703031809
Cattle bredding,Wholesale and retail trade of motor vehicles,7.415051666
Cattle bredding,Wholesale trade,66.6308277
Cattle bredding,Retail trade,1569.525296
Cattle bredding,Accommodation,687.8402149
Cattle bredding,Food and beverage service activities,10639.35466
Cattle bredding,Transport via railways,0.689535268
Cattle bredding,Other passenger land transport,1.130939384
Cattle bredding,Freight transport by road,2.393309338
Cattle bredding,Transport via pipeline,0.079341045
Cattle bredding,Water transport,0.832116165
Cattle bredding,Air transport,15.80781783
Cattle bredding,Warehousing,0.376748896
Cattle bredding,Support activities for land transport,2.578837349
Cattle bredding,Other support activities for transport,10.99497542
Cattle bredding,Postal and courier activities,0.028906422
Cattle bredding,Wireless telecommunications,8.398367799
Cattle bredding,Wired telecommunications,24.03938074
Cattle bredding,Other telecommunications activities,5.399449341
Cattle bredding,Information services activities,121.5660954
Cattle bredding,Publishing activities,4.489903617
Cattle bredding,Banking monetary intermediation,102.1113663
Cattle bredding,Insurance activities,23.37859844
Cattle bredding,Auxiliary financial activities,20.5325369
Cattle bredding,Real estate activities,5.109651655
Cattle bredding,Housing services,0
Cattle bredding,Legal and accounting services,5.771329551
Cattle bredding,Architectural and engineering services,24.04049972
Cattle bredding,Other professional activities,70.89102542
Cattle bredding,Rental and leasing activities,8.065668625
Cattle bredding,Business support activities,41.65898843
Cattle bredding,Public administration,395.8861525
Cattle bredding,Public education,51.73459714
Cattle bredding,Private education,47.18136767
Cattle bredding,Public health activities,208.5151142
Cattle bredding,Private health activities,292.6072894
Cattle bredding,Activities of organizations,434.6513665
Cattle bredding,Artistic and entertainment activities,13.41800308
Cattle bredding,Other personal services,1.222688845
Pigs breeding,Cultivation of annual crops,1.047856148
Pigs breeding,Cultivation of vegetables,0.377049786
Pigs breeding,Cultivation of grapes,0.57408524
Pigs breeding,Cultivation of other fruits,1.50387492
Pigs breeding,Cattle bredding,1.231277204
Pigs breeding,Pigs breeding,0.488503052
Pigs breeding,Chicken breeding,0.898977408
Pigs breeding,Breeding of other animals,0.119683605
Pigs breeding,Farming services,0.079036578
Pigs breeding,Forestry,0.222789927
Pigs breeding,Aquaculture,0.57470643
Pigs breeding,Extractive fishing,0.006485732
Pigs breeding,Coal mining,0.016926173
Pigs breeding,Crude oil and gas mining,0.061517961
Pigs breeding,Copper mining,35.55390991
Pigs breeding,Iron mining,0.027291504
Pigs breeding,Mining of other metals,0.099349395
Pigs breeding,Other mining activities and mining services,0.293594111
Pigs breeding,Processing and preserving of meat,600725.4405
Pigs breeding,Processing and preserving of fishmeal and fish oil,0.902967401
Pigs breeding,Processing and preserving of fish,13.5120821
Pigs breeding,Processing and preserving of fruits and vegetables,6.913894071
Pigs breeding,Manufacture of vegetable and animal oils,1.66523129
Pigs breeding,Manufacture of dairy products,11.0948784
Pigs breeding,Manufacture of grain mill products,6.413830364
Pigs breeding,Manufacture of prepared animal feeds,8.031092368
Pigs breeding,Manufacture of bakery products,8.226155383
Pigs breeding,Manufacture of macaroni noodles and other related products,0.157547295
Pigs breeding,Manufacture of other food products,10.45967506
Pigs breeding,Distilling rectifying and blending of spirits,2.808283915
Pigs breeding,Manufacture of wines,20.01377079
Pigs breeding,Manufacture of beers,9.180029421
Pigs breeding,Manufacture of soft drinks,18.43917197
Pigs breeding,Manufacture of tobacco products,2.41372863
Pigs breeding,Manufacture of textile products,3.238897294
Pigs breeding,Manufacture of wearing apparel,9.073796878
Pigs breeding,Manufacture of leather products,0.221653478
Pigs breeding,Manufacture of footwear,0.712196139
Pigs breeding,Sawmilling and planing of wood,12.71333979
Pigs breeding,Manufacture of wood products,11.37522355
Pigs breeding,Manufacture of pulp and paper,11.25026289
Pigs breeding,Manufacture of containers of paper,4.11384513
Pigs breeding,Manufacture of other paper products,8.650318002
Pigs breeding,Printing,6.201194479
Pigs breeding,Manufacture of refined petroleum products,8.618856078
Pigs breeding,Manufacture of basic chemicals,9.201404278
Pigs breeding,Manufacture of paints,3.229878542
Pigs breeding,Manufacture of pharmaceutical products,26.4743189
Pigs breeding,Manufacture of soap detergents and toilet products,10.91523179
Pigs breeding,Manufacture of other chemical products,5.710988313
Pigs breeding,Manufacture of rubber products,0.148418749
Pigs breeding,Manufacture of plastic products,5.714848367
Pigs breeding,Manufacture of glass and glass products,1.520154191
Pigs breeding,Manufacture of cement,5.209063865
Pigs breeding,Manufacture of concrete and concrete products,14.57353718
Pigs breeding,Manufacture of basic iron and steel,3.387295244
Pigs breeding,Manufacture of other basic metals,1.676098446
Pigs breeding,Manufacture of fabricated metal products,19.08716084
Pigs breeding,Manufacture of industrial and domestic machinery and equipment,8.226463184
Pigs breeding,Manufacture of electric and electronic machinery and equipment,3.028959218
Pigs breeding,Manufacture of transport equipment,5.260687889
Pigs breeding,Manufacture of furniture,5.779694874
Pigs breeding,Repair and installation of machinery and other manufacturing,3.660244294
Pigs breeding,Electric power generation,6.646497283
Pigs breeding,Transmission of electric power,0.104364678
Pigs breeding,Distribution of electric power,1.232417936
Pigs breeding,Gas and steam manufacture and supply,19.75767697
Pigs breeding,Water collection treatment and supply,1.207676331
Pigs breeding,Waste collection and recycling activities,0.868072453
Pigs breeding,Construction of residential buildings,19.46263216
Pigs breeding,Construction of non-residential buildings,6.802203379
Pigs breeding,Civil engineering,0.968487574
Pigs breeding,Specialized construction activities,0.495422508
Pigs breeding,Wholesale and retail trade of motor vehicles,16.55732296
Pigs breeding,Wholesale trade,106.5775475
Pigs breeding,Retail trade,70.24786579
Pigs breeding,Accommodation,1.527083003
Pigs breeding,Food and beverage service activities,6.81156971
Pigs breeding,Transport via railways,1.174614832
Pigs breeding,Other passenger land transport,2.508454707
Pigs breeding,Freight transport by road,3.36884869
Pigs breeding,Transport via pipeline,0.012488962
Pigs breeding,Water transport,0.684079663
Pigs breeding,Air transport,8.158582804
Pigs breeding,Warehousing,2.856276904
Pigs breeding,Support activities for land transport,4.99954733
Pigs breeding,Other support activities for transport,2.704158125
Pigs breeding,Postal and courier activities,0.366537305
Pigs breeding,Wireless telecommunications,14.10009749
Pigs breeding,Wired telecommunications,6.542391054
Pigs breeding,Other telecommunications activities,17.36569142
Pigs breeding,Information services activities,38.39242134
Pigs breeding,Publishing activities,12.24092792
Pigs breeding,Banking monetary intermediation,56.34499214
Pigs breeding,Insurance activities,29.83358102
Pigs breeding,Auxiliary financial activities,13.90522331
Pigs breeding,Real estate activities,25.10359132
Pigs breeding,Housing services,0
Pigs breeding,Legal and accounting services,18.37998759
Pigs breeding,Architectural and engineering services,37.5365504
Pigs breeding,Other professional activities,82.87252564
Pigs breeding,Rental and leasing activities,12.29271823
Pigs breeding,Business support activities,59.9117068
Pigs breeding,Public administration,25.02062231
Pigs breeding,Public education,12.38854279
Pigs breeding,Private education,25.14145893
Pigs breeding,Public health activities,10.71567527
Pigs breeding,Private health activities,15.92370637
Pigs breeding,Activities of organizations,0.507716818
Pigs breeding,Artistic and entertainment activities,7.223897903
Pigs breeding,Other personal services,0.616772831
Chicken breeding,Cultivation of annual crops,9.068084839
Chicken breeding,Cultivation of vegetables,0.021043674
Chicken breeding,Cultivation of grapes,0.039329343
Chicken breeding,Cultivation of other fruits,0.074826999
Chicken breeding,Cattle bredding,4645.183719
Chicken breeding,Pigs breeding,3579.659201
Chicken breeding,Chicken breeding,26112.52346
Chicken breeding,Breeding of other animals,9.422660501
Chicken breeding,Farming services,0.014408433
Chicken breeding,Forestry,0.03638982
Chicken breeding,Aquaculture,4.418686151
Chicken breeding,Extractive fishing,0.007231214
Chicken breeding,Coal mining,0
Chicken breeding,Crude oil and gas mining,0.028469894
Chicken breeding,Copper mining,7.30183244
Chicken breeding,Iron mining,0.002787053
Chicken breeding,Mining of other metals,0.015915338
Chicken breeding,Other mining activities and mining services,0.010870913
Chicken breeding,Processing and preserving of meat,720043.6458
Chicken breeding,Processing and preserving of fishmeal and fish oil,10.08785268
Chicken breeding,Processing and preserving of fish,0.07032055
Chicken breeding,Processing and preserving of fruits and vegetables,0.787237878
Chicken breeding,Manufacture of vegetable and animal oils,547.7949255
Chicken breeding,Manufacture of dairy products,154817.1864
Chicken breeding,Manufacture of grain mill products,7.310011397
Chicken breeding,Manufacture of prepared animal feeds,0.374510383
Chicken breeding,Manufacture of bakery products,1715.402119
Chicken breeding,Manufacture of macaroni noodles and other related products,0.017277078
Chicken breeding,Manufacture of other food products,2905.525704
Chicken breeding,Distilling rectifying and blending of spirits,0.008203699
Chicken breeding,Manufacture of wines,1082.458614
Chicken breeding,Manufacture of beers,0.305735701
Chicken breeding,Manufacture of soft drinks,2.184465877
Chicken breeding,Manufacture of tobacco products,0.000813797
Chicken breeding,Manufacture of textile products,3121.629497
Chicken breeding,Manufacture of wearing apparel,1.365469377
Chicken breeding,Manufacture of leather products,366.6902561
Chicken breeding,Manufacture of footwear,0.299634749
Chicken breeding,Sawmilling and planing of wood,1.115574071
Chicken breeding,Manufacture of wood products,0.484023498
Chicken breeding,Manufacture of pulp and paper,0.861181149
Chicken breeding,Manufacture of containers of paper,0.14568399
Chicken breeding,Manufacture of other paper products,1.039440515
Chicken breeding,Printing,0.612444344
Chicken breeding,Manufacture of refined petroleum products,1.188363735
Chicken breeding,Manufacture of basic chemicals,1.034569294
Chicken breeding,Manufacture of paints,0.312490205
Chicken breeding,Manufacture of pharmaceutical products,152.3033886
Chicken breeding,Manufacture of soap detergents and toilet products,4.130998971
Chicken breeding,Manufacture of other chemical products,0.083570448
Chicken breeding,Manufacture of rubber products,0.04519624
Chicken breeding,Manufacture of plastic products,0.125739422
Chicken breeding,Manufacture of glass and glass products,0.282464588
Chicken breeding,Manufacture of cement,0.258916982
Chicken breeding,Manufacture of concrete and concrete products,1.384023255
Chicken breeding,Manufacture of basic iron and steel,0.03891016
Chicken breeding,Manufacture of other basic metals,0.25854399
Chicken breeding,Manufacture of fabricated metal products,2.810381397
Chicken breeding,Manufacture of industrial and domestic machinery and equipment,0.11684116
Chicken breeding,Manufacture of electric and electronic machinery and equipment,0.488541408
Chicken breeding,Manufacture of transport equipment,0.334359921
Chicken breeding,Manufacture of furniture,3.212593899
Chicken breeding,Repair and installation of machinery and other manufacturing,0.169321817
Chicken breeding,Electric power generation,0.11821772
Chicken breeding,Transmission of electric power,0.046108099
Chicken breeding,Distribution of electric power,0.862586835
Chicken breeding,Gas and steam manufacture and supply,0.119156569
Chicken breeding,Water collection treatment and supply,0.192652612
Chicken breeding,Waste collection and recycling activities,0.289240837
Chicken breeding,Construction of residential buildings,0.157216607
Chicken breeding,Construction of non-residential buildings,0.051227415
Chicken breeding,Civil engineering,0.61719251
Chicken breeding,Specialized construction activities,0.103333727
Chicken breeding,Wholesale and retail trade of motor vehicles,0.950801224
Chicken breeding,Wholesale trade,11.84618789
Chicken breeding,Retail trade,994.853933
Chicken breeding,Accommodation,447.0200725
Chicken breeding,Food and beverage service activities,6920.257495
Chicken breeding,Transport via railways,0.092897025
Chicken breeding,Other passenger land transport,0.145222569
Chicken breeding,Freight transport by road,0.331127779
Chicken breeding,Transport via pipeline,0.012194737
Chicken breeding,Water transport,0.121107889
Chicken breeding,Air transport,2.360069651
Chicken breeding,Warehousing,0.023577167
Chicken breeding,Support activities for land transport,0.339987201
Chicken breeding,Other support activities for transport,1.677982327
Chicken breeding,Postal and courier activities,0
Chicken breeding,Wireless telecommunications,1.133996148
Chicken breeding,Wired telecommunications,3.661002668
Chicken breeding,Other telecommunications activities,0.627186897
Chicken breeding,Information services activities,18.44838331
Chicken breeding,Publishing activities,0.548532113
Chicken breeding,Banking monetary intermediation,15.20025621
Chicken breeding,Insurance activities,3.272294483
Chicken breeding,Auxiliary financial activities,3.024855574
Chicken breeding,Real estate activities,0.487112349
Chicken breeding,Housing services,0
Chicken breeding,Legal and accounting services,0.672614267
Chicken breeding,Architectural and engineering services,3.280763679
Chicken breeding,Other professional activities,10.28087285
Chicken breeding,Rental and leasing activities,1.104400678
Chicken breeding,Business support activities,5.748142031
Chicken breeding,Public administration,230.7391424
Chicken breeding,Public education,32.59371253
Chicken breeding,Private education,27.93311012
Chicken breeding,Public health activities,127.1676227
Chicken breeding,Private health activities,188.6424324
Chicken breeding,Activities of organizations,284.9850813
Chicken breeding,Artistic and entertainment activities,5.407394619
Chicken breeding,Other personal services,0.182719684
Breeding of other animals,Cultivation of annual crops,3.421602333
Breeding of other animals,Cultivation of vegetables,0.027002581
Breeding of other animals,Cultivation of grapes,0.044226772
Breeding of other animals,Cultivation of other fruits,0.103810883
Breeding of other animals,Cattle bredding,1726.364036
Breeding of other animals,Pigs breeding,1330.343176
Breeding of other animals,Chicken breeding,9680.975225
Breeding of other animals,Breeding of other animals,3.507593864
Breeding of other animals,Farming services,0.009930558
Breeding of other animals,Forestry,0.026187776
Breeding of other animals,Aquaculture,1.669621966
Breeding of other animals,Extractive fishing,0.003398657
Breeding of other animals,Coal mining,0.00080866
Breeding of other animals,Crude oil and gas mining,0.015099914
Breeding of other animals,Copper mining,4.817574357
Breeding of other animals,Iron mining,0.002494355
Breeding of other animals,Mining of other metals,0.011544688
Breeding of other animals,Other mining activities and mining services,0.018670155
Breeding of other animals,Processing and preserving of meat,8693.506585
Breeding of other animals,Processing and preserving of fishmeal and fish oil,3.793034512
Breeding of other animals,Processing and preserving of fish,0.675586687
Breeding of other animals,Processing and preserving of fruits and vegetables,0.623683861
Breeding of other animals,Manufacture of vegetable and animal oils,203.6587809
Breeding of other animals,Manufacture of dairy products,57535.8096
Breeding of other animals,Manufacture of grain mill products,3.059908876
Breeding of other animals,Manufacture of prepared animal feeds,0.543662447
Breeding of other animals,Manufacture of bakery products,637.9581806
Breeding of other animals,Manufacture of macaroni noodles and other related products,0.014906799
Breeding of other animals,Manufacture of other food products,1080.37091
Breeding of other animals,Distilling rectifying and blending of spirits,0.137671972
Breeding of other animals,Manufacture of wines,403.3206462
Breeding of other animals,Manufacture of beers,0.569176794
Breeding of other animals,Manufacture of soft drinks,1.814034213
Breeding of other animals,Manufacture of tobacco products,0.11566523
Breeding of other animals,Manufacture of textile products,1160.280721
Breeding of other animals,Manufacture of wearing apparel,1.016764237
Breeding of other animals,Manufacture of leather products,136.2848286
Breeding of other animals,Manufacture of footwear,0.145855294
Breeding of other animals,Sawmilling and planing of wood,1.083903814
Breeding of other animals,Manufacture of wood products,0.750208989
Breeding of other animals,Manufacture of pulp and paper,0.905340918
Breeding of other animals,Manufacture of containers of paper,0.258770484
Breeding of other animals,Manufacture of other paper products,0.857269759
Breeding of other animals,Printing,0.5578707
Breeding of other animals,Manufacture of refined petroleum products,0.91937885
Breeding of other animals,Manufacture of basic chemicals,0.88151757
Breeding of other animals,Manufacture of paints,0.287789233
Breeding of other animals,Manufacture of pharmaceutical products,57.97302943
Breeding of other animals,Manufacture of soap detergents and toilet products,2.138022844
Breeding of other animals,Manufacture of other chemical products,0.308543491
Breeding of other animals,Manufacture of rubber products,0.026396281
Breeding of other animals,Manufacture of plastic products,0.326740288
Breeding of other animals,Manufacture of glass and glass products,0.193280551
Breeding of other animals,Manufacture of cement,0.359462555
Breeding of other animals,Manufacture of concrete and concrete products,1.257138837
Breeding of other animals,Manufacture of basic iron and steel,0.178450848
Breeding of other animals,Manufacture of other basic metals,0.190513272
Breeding of other animals,Manufacture of fabricated metal products,2.112350707
Breeding of other animals,Manufacture of industrial and domestic machinery and equipment,0.442933617
Breeding of other animals,Manufacture of electric and electronic machinery and equipment,0.35339003
Breeding of other animals,Manufacture of transport equipment,0.394154207
Breeding of other animals,Manufacture of furniture,1.568538436
Breeding of other animals,Repair and installation of machinery and other manufacturing,0.247196221
Breeding of other animals,Electric power generation,0.368037606
Breeding of other animals,Transmission of electric power,0.024681065
Breeding of other animals,Distribution of electric power,0.427331616
Breeding of other animals,Gas and steam manufacture and supply,0.99483466
Breeding of other animals,Water collection treatment and supply,0.13998872
Breeding of other animals,Waste collection and recycling activities,0.165021337
Breeding of other animals,Construction of residential buildings,0.996995939
Breeding of other animals,Construction of non-residential buildings,0.346861824
Breeding of other animals,Civil engineering,0.309902537
Breeding of other animals,Specialized construction activities,0.067807928
Breeding of other animals,Wholesale and retail trade of motor vehicles,1.197170532
Breeding of other animals,Wholesale trade,9.9487206
Breeding of other animals,Retail trade,373.4743802
Breeding of other animals,Accommodation,166.2083855
Breeding of other animals,Food and beverage service activities,2572.158386
Breeding of other animals,Transport via railways,0.095798816
Breeding of other animals,Other passenger land transport,0.181874709
Breeding of other animals,Freight transport by road,0.302389628
Breeding of other animals,Transport via pipeline,0.005805623
Breeding of other animals,Water transport,0.084413351
Breeding of other animals,Air transport,1.397880426
Breeding of other animals,Warehousing,0.146531613
Breeding of other animals,Support activities for land transport,0.384081708
Breeding of other animals,Other support activities for transport,0.845939352
Breeding of other animals,Postal and courier activities,0.017511583
Breeding of other animals,Wireless telecommunications,1.158026107
Breeding of other animals,Wired telecommunications,1.876356222
Breeding of other animals,Other telecommunications activities,1.097559807
Breeding of other animals,Information services activities,9.714409604
Breeding of other animals,Publishing activities,0.819123342
Breeding of other animals,Banking monetary intermediation,9.18467662
Breeding of other animals,Insurance activities,2.823073753
Breeding of other animals,Auxiliary financial activities,1.956392066
Breeding of other animals,Real estate activities,1.407411015
Breeding of other animals,Housing services,0
Breeding of other animals,Legal and accounting services,1.165422731
Breeding of other animals,Architectural and engineering services,3.194706244
Breeding of other animals,Other professional activities,8.331401807
Breeding of other animals,Rental and leasing activities,1.059035628
Breeding of other animals,Business support activities,5.317630347
Breeding of other animals,Public administration,87.39190545
Breeding of other animals,Public education,12.71232416
Breeding of other animals,Private education,11.60778307
Breeding of other animals,Public health activities,47.91051206
Breeding of other animals,Private health activities,70.88277852
Breeding of other animals,Activities of organizations,105.008975
Breeding of other animals,Artistic and entertainment activities,2.406225098
Breeding of other animals,Other personal services,0.107515054
Farming services,Cultivation of annual crops,124688.1975
Farming services,Cultivation of vegetables,6986.573191
Farming services,Cultivation of grapes,54569.58466
Farming services,Cultivation of other fruits,59389.21386
Farming services,Cattle bredding,0.944382287
Farming services,Pigs breeding,0.349793876
Farming services,Chicken breeding,0.634537891
Farming services,Breeding of other animals,0.08907754
Farming services,Farming services,602.9483126
Farming services,Forestry,267339.0286
Farming services,Aquaculture,0.396323499
Farming services,Extractive fishing,0.03162447
Farming services,Coal mining,0.011602266
Farming services,Crude oil and gas mining,0.149173261
Farming services,Copper mining,51.81505334
Farming services,Iron mining,0.029182543
Farming services,Mining of other metals,0.127918627
Farming services,Other mining activities and mining services,0.242106581
Farming services,Processing and preserving of meat,14.79462412
Farming services,Processing and preserving of fishmeal and fish oil,0.680918411
Farming services,Processing and preserving of fish,9.52633523
Farming services,Processing and preserving of fruits and vegetables,4.793652828
Farming services,Manufacture of vegetable and animal oils,1.177482647
Farming services,Manufacture of dairy products,14.27874389
Farming services,Manufacture of grain mill products,6.890759127
Farming services,Manufacture of prepared animal feeds,6.91262599
Farming services,Manufacture of bakery products,10.0443874
Farming services,Manufacture of macaroni noodles and other related products,0.17292928
Farming services,Manufacture of other food products,12.7114615
Farming services,Distilling rectifying and blending of spirits,1.955808771
Farming services,Manufacture of wines,19.61520589
Farming services,Manufacture of beers,7.441688161
Farming services,Manufacture of soft drinks,20.84975341
Farming services,Manufacture of tobacco products,1.657580519
Farming services,Manufacture of textile products,3.935312882
Farming services,Manufacture of wearing apparel,11.35191119
Farming services,Manufacture of leather products,0.153684353
Farming services,Manufacture of footwear,0.520373118
Farming services,Sawmilling and planing of wood,12.90744428
Farming services,Manufacture of wood products,9.616512818
Farming services,Manufacture of pulp and paper,10.94841549
Farming services,Manufacture of containers of paper,3.36744666
Farming services,Manufacture of other paper products,9.836242583
Farming services,Printing,6.55258065
Farming services,Manufacture of refined petroleum products,10.37440908
Farming services,Manufacture of basic chemicals,10.19568324
Farming services,Manufacture of paints,3.388466479
Farming services,Manufacture of pharmaceutical products,25.40694808
Farming services,Manufacture of soap detergents and toilet products,12.98790842
Farming services,Manufacture of other chemical products,4.228773677
Farming services,Manufacture of rubber products,0.271607024
Farming services,Manufacture of plastic products,4.389912983
Farming services,Manufacture of glass and glass products,2.103661374
Farming services,Manufacture of cement,4.543768325
Farming services,Manufacture of concrete and concrete products,13.13978454
Farming services,Manufacture of basic iron and steel,2.468110828
Farming services,Manufacture of other basic metals,2.1206491
Farming services,Manufacture of fabricated metal products,23.64644852
Farming services,Manufacture of industrial and domestic machinery and equipment,6.078087795
Farming services,Manufacture of electric and electronic machinery and equipment,3.912437546
Farming services,Manufacture of transport equipment,4.862709289
Farming services,Manufacture of furniture,10.63077941
Farming services,Repair and installation of machinery and other manufacturing,3.145363533
Farming services,Electric power generation,5.000253363
Farming services,Transmission of electric power,0.244836835
Farming services,Distribution of electric power,4.086836045
Farming services,Gas and steam manufacture and supply,13.99101154
Farming services,Water collection treatment and supply,1.551908273
Farming services,Waste collection and recycling activities,1.682152161
Farming services,Construction of residential buildings,13.93181904
Farming services,Construction of non-residential buildings,4.855199027
Farming services,Civil engineering,2.983599438
Farming services,Specialized construction activities,0.727976816
Farming services,Wholesale and retail trade of motor vehicles,14.92304899
Farming services,Wholesale trade,103.8239356
Farming services,Retail trade,75.07462217
Farming services,Accommodation,1.588483199
Farming services,Food and beverage service activities,7.150836985
Farming services,Transport via railways,1.154311407
Farming services,Other passenger land transport,2.265276545
Farming services,Freight transport by road,3.553775275
Farming services,Transport via pipeline,0.054395013
Farming services,Water transport,0.924099061
Farming services,Air transport,14.46280136
Farming services,Warehousing,2.046487843
Farming services,Support activities for land transport,4.704857383
Farming services,Other support activities for transport,8.160348552
Farming services,Postal and courier activities,0.251247787
Farming services,Wireless telecommunications,13.92725541
Farming services,Wired telecommunications,18.24455823
Farming services,Other telecommunications activities,14.26084134
Farming services,Information services activities,95.65540127
Farming services,Publishing activities,10.4523788
Farming services,Banking monetary intermediation,95.7530403
Farming services,Insurance activities,32.74883719
Farming services,Auxiliary financial activities,20.90052947
Farming services,Real estate activities,19.03841149
Farming services,Housing services,0
Farming services,Legal and accounting services,15.12684407
Farming services,Architectural and engineering services,38.06076997
Farming services,Other professional activities,94.13740412
Farming services,Rental and leasing activities,12.57712819
Farming services,Business support activities,62.67183056
Farming services,Public administration,10136.4545
Farming services,Public education,9.00320457
Farming services,Private education,18.97793998
Farming services,Public health activities,16.74739562
Farming services,Private health activities,12.01543912
Farming services,Activities of organizations,0.711087355
Farming services,Artistic and entertainment activities,8.440587717
Farming services,Other personal services,1.109532534
Forestry,Cultivation of annual crops,124045.8768
Forestry,Cultivation of vegetables,7828.441169
Forestry,Cultivation of grapes,53618.70405
Forestry,Cultivation of other fruits,58585.7631
Forestry,Cattle bredding,6197.1869
Forestry,Pigs breeding,605.5616545
Forestry,Chicken breeding,2441.74004
Forestry,Breeding of other animals,501.3655513
Forestry,Farming services,617.1124285
Forestry,Forestry,263047.651
Forestry,Aquaculture,1.986032775
Forestry,Extractive fishing,4.268610057
Forestry,Coal mining,29.88493697
Forestry,Crude oil and gas mining,0.380380298
Forestry,Copper mining,165.7701359
Forestry,Iron mining,0.110599357
Forestry,Mining of other metals,0.436700525
Forestry,Other mining activities and mining services,1.076891496
Forestry,Processing and preserving of meat,1483.546065
Forestry,Processing and preserving of fishmeal and fish oil,3.211930659
Forestry,Processing and preserving of fish,4793.702986
Forestry,Processing and preserving of fruits and vegetables,1167.442837
Forestry,Manufacture of vegetable and animal oils,2125.951224
Forestry,Manufacture of dairy products,1714.431951
Forestry,Manufacture of grain mill products,6593.764999
Forestry,Manufacture of prepared animal feeds,7290.832266
Forestry,Manufacture of bakery products,5599.215914
Forestry,Manufacture of macaroni noodles and other related products,0.91480971
Forestry,Manufacture of other food products,2735.056893
Forestry,Distilling rectifying and blending of spirits,273.6849187
Forestry,Manufacture of wines,264.4822951
Forestry,Manufacture of beers,94.48547645
Forestry,Manufacture of soft drinks,173.752835
Forestry,Manufacture of tobacco products,179.0874989
Forestry,Manufacture of textile products,89.26185998
Forestry,Manufacture of wearing apparel,51.39872599
Forestry,Manufacture of leather products,1.410731288
Forestry,Manufacture of footwear,200.3485496
Forestry,Sawmilling and planing of wood,403868.868
Forestry,Manufacture of wood products,89158.3475
Forestry,Manufacture of pulp and paper,371587.0721
Forestry,Manufacture of containers of paper,15.05019631
Forestry,Manufacture of other paper products,35.9775147
Forestry,Printing,30.55833323
Forestry,Manufacture of refined petroleum products,36.74878939
Forestry,Manufacture of basic chemicals,1178.21623
Forestry,Manufacture of paints,12.98664593
Forestry,Manufacture of pharmaceutical products,756.1369704
Forestry,Manufacture of soap detergents and toilet products,115.6913278
Forestry,Manufacture of other chemical products,371.2592969
Forestry,Manufacture of rubber products,0.778934084
Forestry,Manufacture of plastic products,652.1367344
Forestry,Manufacture of glass and glass products,6.912055593
Forestry,Manufacture of cement,19.49678843
Forestry,Manufacture of concrete and concrete products,131.6187304
Forestry,Manufacture of basic iron and steel,23.2404982
Forestry,Manufacture of other basic metals,9.674198262
Forestry,Manufacture of fabricated metal products,155.4476626
Forestry,Manufacture of industrial and domestic machinery and equipment,395.9014886
Forestry,Manufacture of electric and electronic machinery and equipment,30.47532875
Forestry,Manufacture of transport equipment,61.30056543
Forestry,Manufacture of furniture,5643.319505
Forestry,Repair and installation of machinery and other manufacturing,17.04753468
Forestry,Electric power generation,35718.74939
Forestry,Transmission of electric power,0.632369082
Forestry,Distribution of electric power,9.346779189
Forestry,Gas and steam manufacture and supply,68.85243007
Forestry,Water collection treatment and supply,5.30366988
Forestry,Waste collection and recycling activities,4.70291709
Forestry,Construction of residential buildings,122.1752138
Forestry,Construction of non-residential buildings,32.55505226
Forestry,Civil engineering,83.226325
Forestry,Specialized construction activities,40.83391342
Forestry,Wholesale and retail trade of motor vehicles,62.72682572
Forestry,Wholesale trade,675.9235071
Forestry,Retail trade,284.6168117
Forestry,Accommodation,6.379283969
Forestry,Food and beverage service activities,2033.964504
Forestry,Transport via railways,4.600307598
Forestry,Other passenger land transport,9.510132582
Forestry,Freight transport by road,13.57608912
Forestry,Transport via pipeline,0.115119964
Forestry,Water transport,3.075007322
Forestry,Air transport,42.08329424
Forestry,Warehousing,9.991201852
Forestry,Support activities for land transport,19.25305922
Forestry,Other support activities for transport,19.24024017
Forestry,Postal and courier activities,1.264266214
Forestry,Wireless telecommunications,55.33358574
Forestry,Wired telecommunications,44.19422188
Forestry,Other telecommunications activities,63.60324661
Forestry,Information services activities,241.4112487
Forestry,Publishing activities,45.46216326
Forestry,Banking monetary intermediation,284.1445777
Forestry,Insurance activities,122.2341879
Forestry,Auxiliary financial activities,65.83205222
Forestry,Real estate activities,89.46541379
Forestry,Housing services,0
Forestry,Legal and accounting services,67.37014412
Forestry,Architectural and engineering services,148.8534239
Forestry,Other professional activities,435.0855287
Forestry,Rental and leasing activities,48.92470374
Forestry,Business support activities,431.3787507
Forestry,Public administration,10339.25344
Forestry,Public education,48.11310691
Forestry,Private education,146.8900376
Forestry,Public health activities,72.70455963
Forestry,Private health activities,56.75835184
Forestry,Activities of organizations,6.887358613
Forestry,Artistic and entertainment activities,30.41254776
Forestry,Other personal services,3.206835523
Aquaculture,Cultivation of annual crops,1.167946238
Aquaculture,Cultivation of vegetables,0.468165984
Aquaculture,Cultivation of grapes,0.744387673
Aquaculture,Cultivation of other fruits,1.827850714
Aquaculture,Cattle bredding,1.812826066
Aquaculture,Pigs breeding,0.505679856
Aquaculture,Chicken breeding,0.920011173
Aquaculture,Breeding of other animals,1.906680925
Aquaculture,Farming services,0.141439307
Aquaculture,Forestry,0.380391652
Aquaculture,Aquaculture,243946.9537
Aquaculture,Extractive fishing,0.037807026
Aquaculture,Coal mining,0.016924641
Aquaculture,Crude oil and gas mining,0.184829253
Aquaculture,Copper mining,67.17845501
Aquaculture,Iron mining,0.039361108
Aquaculture,Mining of other metals,0.168277419
Aquaculture,Other mining activities and mining services,0.34065472
Aquaculture,Processing and preserving of meat,69.75548885
Aquaculture,Processing and preserving of fishmeal and fish oil,25345.08777
Aquaculture,Processing and preserving of fish,1990152.209
Aquaculture,Processing and preserving of fruits and vegetables,6.97600019
Aquaculture,Manufacture of vegetable and animal oils,1.706600856
Aquaculture,Manufacture of dairy products,18.78481797
Aquaculture,Manufacture of grain mill products,9.28779992
Aquaculture,Manufacture of prepared animal feeds,10.27691491
Aquaculture,Manufacture of bakery products,24.36148354
Aquaculture,Manufacture of macaroni noodles and other related products,0.232368408
Aquaculture,Manufacture of other food products,186.8325966
Aquaculture,Distilling rectifying and blending of spirits,2.843564009
Aquaculture,Manufacture of wines,26.80733111
Aquaculture,Manufacture of beers,10.50348777
Aquaculture,Manufacture of soft drinks,27.8994801
Aquaculture,Manufacture of tobacco products,2.417035187
Aquaculture,Manufacture of textile products,5.215233722
Aquaculture,Manufacture of wearing apparel,14.98748308
Aquaculture,Manufacture of leather products,0.223649133
Aquaculture,Manufacture of footwear,0.749227567
Aquaculture,Sawmilling and planing of wood,17.54427946
Aquaculture,Manufacture of wood products,13.47073381
Aquaculture,Manufacture of pulp and paper,14.97943635
Aquaculture,Manufacture of containers of paper,4.744500622
Aquaculture,Manufacture of other paper products,13.1518541
Aquaculture,Printing,8.853425573
Aquaculture,Manufacture of refined petroleum products,13.76545339
Aquaculture,Manufacture of basic chemicals,13.6817909
Aquaculture,Manufacture of paints,4.58313227
Aquaculture,Manufacture of pharmaceutical products,34.83838802
Aquaculture,Manufacture of soap detergents and toilet products,17.25948938
Aquaculture,Manufacture of other chemical products,6.072455554
Aquaculture,Manufacture of rubber products,0.344172058
Aquaculture,Manufacture of plastic products,6.258969446
Aquaculture,Manufacture of glass and glass products,2.743507168
Aquaculture,Manufacture of cement,6.330087021
Aquaculture,Manufacture of concrete and concrete products,18.20259392
Aquaculture,Manufacture of basic iron and steel,3.555527457
Aquaculture,Manufacture of other basic metals,2.795825653
Aquaculture,Manufacture of fabricated metal products,31.25855268
Aquaculture,Manufacture of industrial and domestic machinery and equipment,8.731814345
Aquaculture,Manufacture of electric and electronic machinery and equipment,5.144793891
Aquaculture,Manufacture of transport equipment,6.708486174
Aquaculture,Manufacture of furniture,13.46481129
Aquaculture,Repair and installation of machinery and other manufacturing,4.393327666
Aquaculture,Electric power generation,7.157953946
Aquaculture,Transmission of electric power,0.304071674
Aquaculture,Distribution of electric power,4.968586506
Aquaculture,Gas and steam manufacture and supply,20.27201396
Aquaculture,Water collection treatment and supply,2.042038544
Aquaculture,Waste collection and recycling activities,2.120835633
Aquaculture,Construction of residential buildings,20.14185225
Aquaculture,Construction of non-residential buildings,7.023478597
Aquaculture,Civil engineering,3.641758718
Aquaculture,Specialized construction activities,0.942965951
Aquaculture,Wholesale and retail trade of motor vehicles,20.67420407
Aquaculture,Wholesale trade,1146.073237
Aquaculture,Retail trade,101.267863
Aquaculture,Accommodation,248.6591156
Aquaculture,Food and beverage service activities,1056.551188
Aquaculture,Transport via railways,1.576890426
Aquaculture,Other passenger land transport,3.137256813
Aquaculture,Freight transport by road,4.802818174
Aquaculture,Transport via pipeline,0.065309124
Aquaculture,Water transport,1.208594514
Aquaculture,Air transport,18.38044584
Aquaculture,Warehousing,2.958142568
Aquaculture,Support activities for land transport,6.471743708
Aquaculture,Other support activities for transport,9.972065125
Aquaculture,Postal and courier activities,0.366504142
Aquaculture,Wireless telecommunications,19.0107066
Aquaculture,Wired telecommunications,22.39936937
Aquaculture,Other telecommunications activities,20.08076948
Aquaculture,Information services activities,118.2978035
Aquaculture,Publishing activities,14.61577782
Aquaculture,Banking monetary intermediation,122.1795432
Aquaculture,Insurance activities,44.00476915
Aquaculture,Auxiliary financial activities,27.00607521
Aquaculture,Real estate activities,27.21123877
Aquaculture,Housing services,0
Aquaculture,Legal and accounting services,21.29174175
Aquaculture,Architectural and engineering services,51.74372575
Aquaculture,Other professional activities,125.887184
Aquaculture,Rental and leasing activities,17.0752986
Aquaculture,Business support activities,84.80426468
Aquaculture,Public administration,59.83579973
Aquaculture,Public education,12.97668313
Aquaculture,Private education,27.14949942
Aquaculture,Public health activities,21.55017687
Aquaculture,Private health activities,116.6256078
Aquaculture,Activities of organizations,27.5394757
Aquaculture,Artistic and entertainment activities,11.27545842
Aquaculture,Other personal services,1.408164205
Extractive fishing,Cultivation of annual crops,0.147622991
Extractive fishing,Cultivation of vegetables,0.111959839
Extractive fishing,Cultivation of grapes,0.20924611
Extractive fishing,Cultivation of other fruits,0.398106277
Extractive fishing,Cattle bredding,0.142102294
Extractive fishing,Pigs breeding,0.021152541
Extractive fishing,Chicken breeding,0.02593566
Extractive fishing,Breeding of other animals,14.80691925
Extractive fishing,Farming services,0.076657994
Extractive fishing,Forestry,0.193606802
Extractive fishing,Aquaculture,10050.71132
Extractive fishing,Extractive fishing,0.038472634
Extractive fishing,Coal mining,0
Extractive fishing,Crude oil and gas mining,0.151469974
Extractive fishing,Copper mining,38.84834854
Extractive fishing,Iron mining,0.014828116
Extractive fishing,Mining of other metals,0.084675264
Extractive fishing,Other mining activities and mining services,0.057837127
Extractive fishing,Processing and preserving of meat,405.2654904
Extractive fishing,Processing and preserving of fishmeal and fish oil,210813.7925
Extractive fishing,Processing and preserving of fish,247200.5472
Extractive fishing,Processing and preserving of fruits and vegetables,0.077053258
Extractive fishing,Manufacture of vegetable and animal oils,0.050999261
Extractive fishing,Manufacture of dairy products,9.446778152
Extractive fishing,Manufacture of grain mill products,3.530806743
Extractive fishing,Manufacture of prepared animal feeds,7.18602107
Extractive fishing,Manufacture of bakery products,98.22417269
Extractive fishing,Manufacture of macaroni noodles and other related products,0.091920207
Extractive fishing,Manufacture of other food products,1421.808455
Extractive fishing,Distilling rectifying and blending of spirits,0.043646602
Extractive fishing,Manufacture of wines,8.3467476
Extractive fishing,Manufacture of beers,1.626622792
Extractive fishing,Manufacture of soft drinks,11.62213629
Extractive fishing,Manufacture of tobacco products,0.00432969
Extractive fishing,Manufacture of textile products,2.42789202
Extractive fishing,Manufacture of wearing apparel,7.26478329
Extractive fishing,Manufacture of leather products,0.002475894
Extractive fishing,Manufacture of footwear,0.045564816
Extractive fishing,Sawmilling and planing of wood,5.935251279
Extractive fishing,Manufacture of wood products,2.575177355
Extractive fishing,Manufacture of pulp and paper,4.581790351
Extractive fishing,Manufacture of containers of paper,0.775090703
Extractive fishing,Manufacture of other paper products,5.530193655
Extractive fishing,Printing,3.25842198
Extractive fishing,Manufacture of refined petroleum products,6.32251821
Extractive fishing,Manufacture of basic chemicals,5.504277021
Extractive fishing,Manufacture of paints,1.662559158
Extractive fishing,Manufacture of pharmaceutical products,10.27651999
Extractive fishing,Manufacture of soap detergents and toilet products,7.793858341
Extractive fishing,Manufacture of other chemical products,0.444624541
Extractive fishing,Manufacture of rubber products,0.240460088
Extractive fishing,Manufacture of plastic products,0.668978498
Extractive fishing,Manufacture of glass and glass products,1.502812186
Extractive fishing,Manufacture of cement,1.377530534
Extractive fishing,Manufacture of concrete and concrete products,4.459186404
Extractive fishing,Manufacture of basic iron and steel,0.207015905
Extractive fishing,Manufacture of other basic metals,1.375546086
Extractive fishing,Manufacture of fabricated metal products,14.95222972
Extractive fishing,Manufacture of industrial and domestic machinery and equipment,0.621636573
Extractive fishing,Manufacture of electric and electronic machinery and equipment,2.599214245
Extractive fishing,Manufacture of transport equipment,1.778913837
Extractive fishing,Manufacture of furniture,9.440263167
Extractive fishing,Repair and installation of machinery and other manufacturing,0.900852356
Extractive fishing,Electric power generation,0.628960366
Extractive fishing,Transmission of electric power,0.245311503
Extractive fishing,Distribution of electric power,4.589269101
Extractive fishing,Gas and steam manufacture and supply,0.633955374
Extractive fishing,Water collection treatment and supply,1.024980492
Extractive fishing,Waste collection and recycling activities,1.53886424
Extractive fishing,Construction of residential buildings,0.836448332
Extractive fishing,Construction of non-residential buildings,0.272548091
Extractive fishing,Civil engineering,3.283683917
Extractive fishing,Specialized construction activities,0.549772224
Extractive fishing,Wholesale and retail trade of motor vehicles,5.058601063
Extractive fishing,Wholesale trade,8395.263061
Extractive fishing,Retail trade,38.10963872
Extractive fishing,Accommodation,2051.232577
Extractive fishing,Food and beverage service activities,8711.518725
Extractive fishing,Transport via railways,0.494245253
Extractive fishing,Other passenger land transport,0.772635778
Extractive fishing,Freight transport by road,1.761717686
Extractive fishing,Transport via pipeline,0.064880341
Extractive fishing,Water transport,0.644337094
Extractive fishing,Air transport,12.55641089
Extractive fishing,Warehousing,0.125438926
Extractive fishing,Support activities for land transport,1.808852969
Extractive fishing,Other support activities for transport,8.92746346
Extractive fishing,Postal and courier activities,0
Extractive fishing,Wireless telecommunications,6.033263291
Extractive fishing,Wired telecommunications,19.47783776
Extractive fishing,Other telecommunications activities,3.336857616
Extractive fishing,Information services activities,98.15196811
Extractive fishing,Publishing activities,2.918386157
Extractive fishing,Banking monetary intermediation,80.87077539
Extractive fishing,Insurance activities,17.40977182
Extractive fishing,Auxiliary financial activities,16.09330871
Extractive fishing,Real estate activities,2.591611143
Extractive fishing,Housing services,0
Extractive fishing,Legal and accounting services,3.578547395
Extractive fishing,Architectural and engineering services,17.45483095
Extractive fishing,Other professional activities,52.8440721
Extractive fishing,Rental and leasing activities,5.875804848
Extractive fishing,Business support activities,30.58216234
Extractive fishing,Public administration,42.76622842
Extractive fishing,Public education,0.723788994
Extractive fishing,Private education,2.469268226
Extractive fishing,Public health activities,13.30919841
Extractive fishing,Private health activities,828.6655998
Extractive fishing,Activities of organizations,221.8855725
Extractive fishing,Artistic and entertainment activities,5.200631482
Extractive fishing,Other personal services,0.972133781
Coal mining,Cultivation of annual crops,2.813733504
Coal mining,Cultivation of vegetables,431.4536747
Coal mining,Cultivation of grapes,13.73989859
Coal mining,Cultivation of other fruits,45.0633709
Coal mining,Cattle bredding,0.259720048
Coal mining,Pigs breeding,0.020194029
Coal mining,Chicken breeding,0.110533117
Coal mining,Breeding of other animals,0.065465539
Coal mining,Farming services,69.79355916
Coal mining,Forestry,0.016404373
Coal mining,Aquaculture,0.000285871
Coal mining,Extractive fishing,0.0032598
Coal mining,Coal mining,0
Coal mining,Crude oil and gas mining,0.012834104
Coal mining,Copper mining,369.1081599
Coal mining,Iron mining,1052.790104
Coal mining,Mining of other metals,0.007174565
Coal mining,Other mining activities and mining services,40.44475962
Coal mining,Processing and preserving of meat,55.30900806
Coal mining,Processing and preserving of fishmeal and fish oil,187.1823447
Coal mining,Processing and preserving of fish,7.269032438
Coal mining,Processing and preserving of fruits and vegetables,489.6060739
Coal mining,Manufacture of vegetable and animal oils,0.004321185
Coal mining,Manufacture of dairy products,669.2085463
Coal mining,Manufacture of grain mill products,0.379324178
Coal mining,Manufacture of prepared animal feeds,0.4359675
Coal mining,Manufacture of bakery products,21.52179333
Coal mining,Manufacture of macaroni noodles and other related products,0.007788432
Coal mining,Manufacture of other food products,2009.98164
Coal mining,Distilling rectifying and blending of spirits,0.003698192
Coal mining,Manufacture of wines,65.35650327
Coal mining,Manufacture of beers,638.5227713
Coal mining,Manufacture of soft drinks,15.94392782
Coal mining,Manufacture of tobacco products,0.000366856
Coal mining,Manufacture of textile products,0.399500226
Coal mining,Manufacture of wearing apparel,0.615547656
Coal mining,Manufacture of leather products,0.244282749
Coal mining,Manufacture of footwear,0.003860723
Coal mining,Sawmilling and planing of wood,0.502895939
Coal mining,Manufacture of wood products,0.218195688
Coal mining,Manufacture of pulp and paper,0.388216716
Coal mining,Manufacture of containers of paper,0.065673709
Coal mining,Manufacture of other paper products,0.468575264
Coal mining,Printing,0.276087246
Coal mining,Manufacture of refined petroleum products,0.535709203
Coal mining,Manufacture of basic chemicals,337.1688309
Coal mining,Manufacture of paints,0.140869225
Coal mining,Manufacture of pharmaceutical products,1.330583524
Coal mining,Manufacture of soap detergents and toilet products,1.444084632
Coal mining,Manufacture of other chemical products,2.070769293
Coal mining,Manufacture of rubber products,0.020374268
Coal mining,Manufacture of plastic products,0.104502407
Coal mining,Manufacture of glass and glass products,205.6485718
Coal mining,Manufacture of cement,1314.071758
Coal mining,Manufacture of concrete and concrete products,478.8836988
Coal mining,Manufacture of basic iron and steel,6655.94536
Coal mining,Manufacture of other basic metals,196.3116401
Coal mining,Manufacture of fabricated metal products,8.974363862
Coal mining,Manufacture of industrial and domestic machinery and equipment,69.80746676
Coal mining,Manufacture of electric and electronic machinery and equipment,1.618530039
Coal mining,Manufacture of transport equipment,0.743947348
Coal mining,Manufacture of furniture,0.799876836
Coal mining,Repair and installation of machinery and other manufacturing,0.737116834
Coal mining,Electric power generation,8469.236963
Coal mining,Transmission of electric power,0.02078533
Coal mining,Distribution of electric power,0.388850393
Coal mining,Gas and steam manufacture and supply,0.053715263
Coal mining,Water collection treatment and supply,0.971705039
Coal mining,Waste collection and recycling activities,0.130388511
Coal mining,Construction of residential buildings,0.292920546
Coal mining,Construction of non-residential buildings,7.543567152
Coal mining,Civil engineering,76.89632266
Coal mining,Specialized construction activities,14.86765785
Coal mining,Wholesale and retail trade of motor vehicles,5.179859179
Coal mining,Wholesale trade,3.690408776
Coal mining,Retail trade,4.648160486
Coal mining,Accommodation,0.064974037
Coal mining,Food and beverage service activities,0.297660333
Coal mining,Transport via railways,0.041877575
Coal mining,Other passenger land transport,0.065465703
Coal mining,Freight transport by road,0.163222087
Coal mining,Transport via pipeline,0.005497334
Coal mining,Water transport,0.05459491
Coal mining,Air transport,1.063909134
Coal mining,Warehousing,0.010628485
Coal mining,Support activities for land transport,0.153264752
Coal mining,Other support activities for transport,0.756427135
Coal mining,Postal and courier activities,0
Coal mining,Wireless telecommunications,0.511200532
Coal mining,Wired telecommunications,1.65036408
Coal mining,Other telecommunications activities,0.282733125
Coal mining,Information services activities,8.316450958
Coal mining,Publishing activities,0.247275891
Coal mining,Banking monetary intermediation,6.85220939
Coal mining,Insurance activities,1.475136121
Coal mining,Auxiliary financial activities,1.363591737
Coal mining,Real estate activities,0.219588128
Coal mining,Housing services,0
Coal mining,Legal and accounting services,0.303211586
Coal mining,Architectural and engineering services,1.478953997
Coal mining,Other professional activities,4.477496911
Coal mining,Rental and leasing activities,0.497859022
Coal mining,Business support activities,2.591237427
Coal mining,Public administration,3.976348752
Coal mining,Public education,0.061326897
Coal mining,Private education,0.21387407
Coal mining,Public health activities,1.127693087
Coal mining,Private health activities,0.131972418
Coal mining,Activities of organizations,0.04354592
Coal mining,Artistic and entertainment activities,0.418453029
Coal mining,Other personal services,0.082369239
Crude oil and gas mining,Cultivation of annual crops,55.2338419
Crude oil and gas mining,Cultivation of vegetables,60.28918589
Crude oil and gas mining,Cultivation of grapes,270.2192466
Crude oil and gas mining,Cultivation of other fruits,886.3854175
Crude oil and gas mining,Cattle bredding,4.989941365
Crude oil and gas mining,Pigs breeding,0.407541787
Crude oil and gas mining,Chicken breeding,2.153268057
Crude oil and gas mining,Breeding of other animals,1.292700318
Crude oil and gas mining,Farming services,208.1988745
Crude oil and gas mining,Forestry,0.216624802
Crude oil and gas mining,Aquaculture,0.14066139
Crude oil and gas mining,Extractive fishing,0.026241365
Crude oil and gas mining,Coal mining,0.697434738
Crude oil and gas mining,Crude oil and gas mining,31.59521386
Crude oil and gas mining,Copper mining,686.8817122
Crude oil and gas mining,Iron mining,91.36749549
Crude oil and gas mining,Mining of other metals,57.55310091
Crude oil and gas mining,Other mining activities and mining services,62.84660898
Crude oil and gas mining,Processing and preserving of meat,4.450288744
Crude oil and gas mining,Processing and preserving of fishmeal and fish oil,0.277147955
Crude oil and gas mining,Processing and preserving of fish,72.59366086
Crude oil and gas mining,Processing and preserving of fruits and vegetables,0.086911998
Crude oil and gas mining,Manufacture of vegetable and animal oils,0.67736648
Crude oil and gas mining,Manufacture of dairy products,13.33764478
Crude oil and gas mining,Manufacture of grain mill products,7.415872366
Crude oil and gas mining,Manufacture of prepared animal feeds,7.506909691
Crude oil and gas mining,Manufacture of bakery products,6.291271986
Crude oil and gas mining,Manufacture of macaroni noodles and other related products,0.061312942
Crude oil and gas mining,Manufacture of other food products,33.94920813
Crude oil and gas mining,Distilling rectifying and blending of spirits,6.624273355
Crude oil and gas mining,Manufacture of wines,12.04459904
Crude oil and gas mining,Manufacture of beers,3.018762533
Crude oil and gas mining,Manufacture of soft drinks,7.681341953
Crude oil and gas mining,Manufacture of tobacco products,3.458020053
Crude oil and gas mining,Manufacture of textile products,6.68282992
Crude oil and gas mining,Manufacture of wearing apparel,9.302738253
Crude oil and gas mining,Manufacture of leather products,0.012129987
Crude oil and gas mining,Manufacture of footwear,0.551204238
Crude oil and gas mining,Sawmilling and planing of wood,6.589770016
Crude oil and gas mining,Manufacture of wood products,2.532406902
Crude oil and gas mining,Manufacture of pulp and paper,5.925574428
Crude oil and gas mining,Manufacture of containers of paper,0.619918495
Crude oil and gas mining,Manufacture of other paper products,5.947271696
Crude oil and gas mining,Printing,7.862975491
Crude oil and gas mining,Manufacture of refined petroleum products,61827.48232
Crude oil and gas mining,Manufacture of basic chemicals,13565.94903
Crude oil and gas mining,Manufacture of paints,1.442629047
Crude oil and gas mining,Manufacture of pharmaceutical products,22.18143554
Crude oil and gas mining,Manufacture of soap detergents and toilet products,20.57272524
Crude oil and gas mining,Manufacture of other chemical products,42.28234745
Crude oil and gas mining,Manufacture of rubber products,0.363710968
Crude oil and gas mining,Manufacture of plastic products,1.986952599
Crude oil and gas mining,Manufacture of glass and glass products,113.9680147
Crude oil and gas mining,Manufacture of cement,2349.203051
Crude oil and gas mining,Manufacture of concrete and concrete products,4852.960866
Crude oil and gas mining,Manufacture of basic iron and steel,101.5255585
Crude oil and gas mining,Manufacture of other basic metals,6.421298599
Crude oil and gas mining,Manufacture of fabricated metal products,286.2885448
Crude oil and gas mining,Manufacture of industrial and domestic machinery and equipment,98.76419204
Crude oil and gas mining,Manufacture of electric and electronic machinery and equipment,46.77562007
Crude oil and gas mining,Manufacture of transport equipment,2.959479475
Crude oil and gas mining,Manufacture of furniture,13.07438556
Crude oil and gas mining,Repair and installation of machinery and other manufacturing,13.86262956
Crude oil and gas mining,Electric power generation,840.5789587
Crude oil and gas mining,Transmission of electric power,0.668599901
Crude oil and gas mining,Distribution of electric power,3.483338925
Crude oil and gas mining,Gas and steam manufacture and supply,23280.21536
Crude oil and gas mining,Water collection treatment and supply,8.274034466
Crude oil and gas mining,Waste collection and recycling activities,2.128202931
Crude oil and gas mining,Construction of residential buildings,23.59478147
Crude oil and gas mining,Construction of non-residential buildings,158.0012216
Crude oil and gas mining,Civil engineering,1662.141429
Crude oil and gas mining,Specialized construction activities,361.0400634
Crude oil and gas mining,Wholesale and retail trade of motor vehicles,9.161064056
Crude oil and gas mining,Wholesale trade,125.7098059
Crude oil and gas mining,Retail trade,40.06813412
Crude oil and gas mining,Accommodation,2.925231264
Crude oil and gas mining,Food and beverage service activities,2.95688376
Crude oil and gas mining,Transport via railways,0.374811559
Crude oil and gas mining,Other passenger land transport,10.3908689
Crude oil and gas mining,Freight transport by road,19.33027405
Crude oil and gas mining,Transport via pipeline,152.0173112
Crude oil and gas mining,Water transport,1.024128495
Crude oil and gas mining,Air transport,9.297403415
Crude oil and gas mining,Warehousing,0.082905523
Crude oil and gas mining,Support activities for land transport,1.935012963
Crude oil and gas mining,Other support activities for transport,11.65216512
Crude oil and gas mining,Postal and courier activities,0.97601822
Crude oil and gas mining,Wireless telecommunications,3.987524951
Crude oil and gas mining,Wired telecommunications,12.87335896
Crude oil and gas mining,Other telecommunications activities,5.330754468
Crude oil and gas mining,Information services activities,65.6700639
Crude oil and gas mining,Publishing activities,4.950106431
Crude oil and gas mining,Banking monetary intermediation,53.44938868
Crude oil and gas mining,Insurance activities,17.91333917
Crude oil and gas mining,Auxiliary financial activities,10.63644448
Crude oil and gas mining,Real estate activities,18.37700644
Crude oil and gas mining,Housing services,0
Crude oil and gas mining,Legal and accounting services,3.962849849
Crude oil and gas mining,Architectural and engineering services,343.6487387
Crude oil and gas mining,Other professional activities,65.50717618
Crude oil and gas mining,Rental and leasing activities,10.93416227
Crude oil and gas mining,Business support activities,30.62988459
Crude oil and gas mining,Public administration,44.16953558
Crude oil and gas mining,Public education,0.539990953
Crude oil and gas mining,Private education,1.815690084
Crude oil and gas mining,Public health activities,9.44065814
Crude oil and gas mining,Private health activities,1.329934546
Crude oil and gas mining,Activities of organizations,1.24449552
Crude oil and gas mining,Artistic and entertainment activities,5.623109191
Crude oil and gas mining,Other personal services,3.306319241
Copper mining,Cultivation of annual crops,7138.212779
Copper mining,Cultivation of vegetables,4615.289162
Copper mining,Cultivation of grapes,10581.6281
Copper mining,Cultivation of other fruits,13408.25893
Copper mining,Cattle bredding,1130.761857
Copper mining,Pigs breeding,123.4730305
Copper mining,Chicken breeding,259.0422264
Copper mining,Breeding of other animals,117.6188634
Copper mining,Farming services,1632.144914
Copper mining,Forestry,762.7122069
Copper mining,Aquaculture,4522.750631
Copper mining,Extractive fishing,119.5368925
Copper mining,Coal mining,6.932934997
Copper mining,Crude oil and gas mining,209.2287421
Copper mining,Copper mining,1452437.612
Copper mining,Iron mining,20552.75517
Copper mining,Mining of other metals,2668.23111
Copper mining,Other mining activities and mining services,1646.440011
Copper mining,Processing and preserving of meat,7772.972849
Copper mining,Processing and preserving of fishmeal and fish oil,303.8844719
Copper mining,Processing and preserving of fish,3877.738131
Copper mining,Processing and preserving of fruits and vegetables,4020.071953
Copper mining,Manufacture of vegetable and animal oils,410.2090088
Copper mining,Manufacture of dairy products,2739.135686
Copper mining,Manufacture of grain mill products,1619.486104
Copper mining,Manufacture of prepared animal feeds,4954.287023
Copper mining,Manufacture of bakery products,2682.346716
Copper mining,Manufacture of macaroni noodles and other related products,39.55649704
Copper mining,Manufacture of other food products,5501.790725
Copper mining,Distilling rectifying and blending of spirits,692.2587269
Copper mining,Manufacture of wines,4933.128042
Copper mining,Manufacture of beers,2278.921812
Copper mining,Manufacture of soft drinks,5267.628591
Copper mining,Manufacture of tobacco products,600.27254
Copper mining,Manufacture of textile products,3617.886023
Copper mining,Manufacture of wearing apparel,2266.92843
Copper mining,Manufacture of leather products,1271.32188
Copper mining,Manufacture of footwear,960.5245159
Copper mining,Sawmilling and planing of wood,3122.767141
Copper mining,Manufacture of wood products,11588.01042
Copper mining,Manufacture of pulp and paper,19529.47558
Copper mining,Manufacture of containers of paper,6071.460882
Copper mining,Manufacture of other paper products,2862.01538
Copper mining,Printing,1684.900986
Copper mining,Manufacture of refined petroleum products,2093.029424
Copper mining,Manufacture of basic chemicals,98343.25371
Copper mining,Manufacture of paints,1973.554466
Copper mining,Manufacture of pharmaceutical products,8718.391556
Copper mining,Manufacture of soap detergents and toilet products,4914.979348
Copper mining,Manufacture of other chemical products,20982.78438
Copper mining,Manufacture of rubber products,3960.864564
Copper mining,Manufacture of plastic products,24577.40504
Copper mining,Manufacture of glass and glass products,2879.555981
Copper mining,Manufacture of cement,19767.5584
Copper mining,Manufacture of concrete and concrete products,37407.28463
Copper mining,Manufacture of basic iron and steel,2539.820455
Copper mining,Manufacture of other basic metals,194499.7123
Copper mining,Manufacture of fabricated metal products,8842.886989
Copper mining,Manufacture of industrial and domestic machinery and equipment,4941.147035
Copper mining,Manufacture of electric and electronic machinery and equipment,10463.49514
Copper mining,Manufacture of transport equipment,1446.159174
Copper mining,Manufacture of furniture,1557.026478
Copper mining,Repair and installation of machinery and other manufacturing,3153.607899
Copper mining,Electric power generation,3114.052033
Copper mining,Transmission of electric power,114.3027962
Copper mining,Distribution of electric power,611.6790713
Copper mining,Gas and steam manufacture and supply,4909.346736
Copper mining,Water collection treatment and supply,2639.051118
Copper mining,Waste collection and recycling activities,1234.587497
Copper mining,Construction of residential buildings,6011.439101
Copper mining,Construction of non-residential buildings,3027.877546
Copper mining,Civil engineering,13300.40045
Copper mining,Specialized construction activities,2985.773223
Copper mining,Wholesale and retail trade of motor vehicles,4163.126065
Copper mining,Wholesale trade,27893.50976
Copper mining,Retail trade,20273.50734
Copper mining,Accommodation,452.4969781
Copper mining,Food and beverage service activities,1872.32575
Copper mining,Transport via railways,385.7608946
Copper mining,Other passenger land transport,1136.996941
Copper mining,Freight transport by road,1774.617221
Copper mining,Transport via pipeline,53.00876798
Copper mining,Water transport,254.5318082
Copper mining,Air transport,2086.58615
Copper mining,Warehousing,728.8277418
Copper mining,Support activities for land transport,1251.913346
Copper mining,Other support activities for transport,899.4631123
Copper mining,Postal and courier activities,103.5443074
Copper mining,Wireless telecommunications,3753.966745
Copper mining,Wired telecommunications,1889.73264
Copper mining,Other telecommunications activities,4744.505107
Copper mining,Information services activities,9082.325009
Copper mining,Publishing activities,3053.172193
Copper mining,Banking monetary intermediation,13582.42684
Copper mining,Insurance activities,7291.707311
Copper mining,Auxiliary financial activities,5148.625751
Copper mining,Real estate activities,6401.167195
Copper mining,Housing services,0
Copper mining,Legal and accounting services,4553.860041
Copper mining,Architectural and engineering services,9554.106725
Copper mining,Other professional activities,20350.07563
Copper mining,Rental and leasing activities,3706.56736
Copper mining,Business support activities,15030.95883
Copper mining,Public administration,7478.891899
Copper mining,Public education,3217.254079
Copper mining,Private education,6892.099081
Copper mining,Public health activities,2979.561347
Copper mining,Private health activities,7443.459407
Copper mining,Activities of organizations,683.1577085
Copper mining,Artistic and entertainment activities,1846.903004
Copper mining,Other personal services,248.3725596
Iron mining,Cultivation of annual crops,5.650865416
Iron mining,Cultivation of vegetables,5.589784123
Iron mining,Cultivation of grapes,24.15796142
Iron mining,Cultivation of other fruits,78.74778258
Iron mining,Cattle bredding,1.394397563
Iron mining,Pigs breeding,0.417926311
Iron mining,Chicken breeding,0.848753037
Iron mining,Breeding of other animals,0.213358625
Iron mining,Farming services,19.31248901
Iron mining,Forestry,0.318328637
Iron mining,Aquaculture,38.68997975
Iron mining,Extractive fishing,100.4104878
Iron mining,Coal mining,0.56650446
Iron mining,Crude oil and gas mining,0.720105348
Iron mining,Copper mining,1397.779414
Iron mining,Iron mining,5144.591919
Iron mining,Mining of other metals,75.26613034
Iron mining,Other mining activities and mining services,100.7630248
Iron mining,Processing and preserving of meat,79.65090357
Iron mining,Processing and preserving of fishmeal and fish oil,38.88071189
Iron mining,Processing and preserving of fish,134.0857787
Iron mining,Processing and preserving of fruits and vegetables,87.81275145
Iron mining,Manufacture of vegetable and animal oils,27.98560244
Iron mining,Manufacture of dairy products,40.89979103
Iron mining,Manufacture of grain mill products,24.10890859
Iron mining,Manufacture of prepared animal feeds,16.82592433
Iron mining,Manufacture of bakery products,14.80017409
Iron mining,Manufacture of macaroni noodles and other related products,0.738394472
Iron mining,Manufacture of other food products,52.39829522
Iron mining,Distilling rectifying and blending of spirits,11.18345415
Iron mining,Manufacture of wines,175.3234327
Iron mining,Manufacture of beers,11.97500904
Iron mining,Manufacture of soft drinks,29.47741618
Iron mining,Manufacture of tobacco products,5.520649203
Iron mining,Manufacture of textile products,28.79326311
Iron mining,Manufacture of wearing apparel,40.37500617
Iron mining,Manufacture of leather products,3.293075637
Iron mining,Manufacture of footwear,4.893433301
Iron mining,Sawmilling and planing of wood,179.5099333
Iron mining,Manufacture of wood products,103.5509781
Iron mining,Manufacture of pulp and paper,200.8074014
Iron mining,Manufacture of containers of paper,34.64030917
Iron mining,Manufacture of other paper products,29.14665691
Iron mining,Printing,13.947645
Iron mining,Manufacture of refined petroleum products,41.42614439
Iron mining,Manufacture of basic chemicals,711.9655061
Iron mining,Manufacture of paints,6.069894919
Iron mining,Manufacture of pharmaceutical products,102.2900903
Iron mining,Manufacture of soap detergents and toilet products,48.31432083
Iron mining,Manufacture of other chemical products,40.51935292
Iron mining,Manufacture of rubber products,17.72903505
Iron mining,Manufacture of plastic products,50.52871529
Iron mining,Manufacture of glass and glass products,41.05208847
Iron mining,Manufacture of cement,213.060441
Iron mining,Manufacture of concrete and concrete products,462.2054158
Iron mining,Manufacture of basic iron and steel,62655.19365
Iron mining,Manufacture of other basic metals,19.04437826
Iron mining,Manufacture of fabricated metal products,81.9294706
Iron mining,Manufacture of industrial and domestic machinery and equipment,44.66501102
Iron mining,Manufacture of electric and electronic machinery and equipment,20.81066049
Iron mining,Manufacture of transport equipment,10.51219435
Iron mining,Manufacture of furniture,31.83892346
Iron mining,Repair and installation of machinery and other manufacturing,9.287015343
Iron mining,Electric power generation,63.58555973
Iron mining,Transmission of electric power,0.588485067
Iron mining,Distribution of electric power,3.318069463
Iron mining,Gas and steam manufacture and supply,31.69057492
Iron mining,Water collection treatment and supply,7.370234701
Iron mining,Waste collection and recycling activities,2.582921792
Iron mining,Construction of residential buildings,29.61355725
Iron mining,Construction of non-residential buildings,25.77309532
Iron mining,Civil engineering,255.0141731
Iron mining,Specialized construction activities,80.96372073
Iron mining,Wholesale and retail trade of motor vehicles,325.9746698
Iron mining,Wholesale trade,2117.80511
Iron mining,Retail trade,668.0513032
Iron mining,Accommodation,4.033598765
Iron mining,Food and beverage service activities,7.16404496
Iron mining,Transport via railways,101.186304
Iron mining,Other passenger land transport,487.0974132
Iron mining,Freight transport by road,87.03651709
Iron mining,Transport via pipeline,0.246175893
Iron mining,Water transport,187.2220511
Iron mining,Air transport,656.5032148
Iron mining,Warehousing,17.1167946
Iron mining,Support activities for land transport,9.856019339
Iron mining,Other support activities for transport,615.9206098
Iron mining,Postal and courier activities,6.944231737
Iron mining,Wireless telecommunications,30.33393253
Iron mining,Wired telecommunications,13.50844788
Iron mining,Other telecommunications activities,16.49441517
Iron mining,Information services activities,78.82833003
Iron mining,Publishing activities,13.42439262
Iron mining,Banking monetary intermediation,79.80354599
Iron mining,Insurance activities,34.43591554
Iron mining,Auxiliary financial activities,17.30194495
Iron mining,Real estate activities,32.48960124
Iron mining,Housing services,0
Iron mining,Legal and accounting services,16.21362204
Iron mining,Architectural and engineering services,299.3110145
Iron mining,Other professional activities,108.6434137
Iron mining,Rental and leasing activities,21.64089827
Iron mining,Business support activities,91.11073695
Iron mining,Public administration,46.89277731
Iron mining,Public education,9.607951018
Iron mining,Private education,19.84594798
Iron mining,Public health activities,14.23894558
Iron mining,Private health activities,12.55362799
Iron mining,Activities of organizations,1.316842115
Iron mining,Artistic and entertainment activities,9.319904384
Iron mining,Other personal services,7.362603903
Mining of other metals,Cultivation of annual crops,176.7731875
Mining of other metals,Cultivation of vegetables,192.9299078
Mining of other metals,Cultivation of grapes,864.2981399
Mining of other metals,Cultivation of other fruits,2835.290291
Mining of other metals,Cattle bredding,16.04394497
Mining of other metals,Pigs breeding,1.22599634
Mining of other metals,Chicken breeding,6.901469375
Mining of other metals,Breeding of other animals,4.09900743
Mining of other metals,Farming services,661.3383071
Mining of other metals,Forestry,0.621035574
Mining of other metals,Aquaculture,0.010822488
Mining of other metals,Extractive fishing,0.123409272
Mining of other metals,Coal mining,0
Mining of other metals,Crude oil and gas mining,0.485872612
Mining of other metals,Copper mining,12173.37669
Mining of other metals,Iron mining,70.68092932
Mining of other metals,Mining of other metals,0.271614174
Mining of other metals,Other mining activities and mining services,63.68612817
Mining of other metals,Processing and preserving of meat,5.21418502
Mining of other metals,Processing and preserving of fishmeal and fish oil,0.281373488
Mining of other metals,Processing and preserving of fish,219.1424447
Mining of other metals,Processing and preserving of fruits and vegetables,0.247164942
Mining of other metals,Manufacture of vegetable and animal oils,0.16359113
Mining of other metals,Manufacture of dairy products,52.9919796
Mining of other metals,Manufacture of grain mill products,16.37067724
Mining of other metals,Manufacture of prepared animal feeds,23.20434426
Mining of other metals,Manufacture of bakery products,20.00458802
Mining of other metals,Manufacture of macaroni noodles and other related products,0.29485389
Mining of other metals,Manufacture of other food products,107.1836074
Mining of other metals,Distilling rectifying and blending of spirits,0.140005889
Mining of other metals,Manufacture of wines,26.78718942
Mining of other metals,Manufacture of beers,5.217743434
Mining of other metals,Manufacture of soft drinks,37.2805088
Mining of other metals,Manufacture of tobacco products,0.013888413
Mining of other metals,Manufacture of textile products,19.98410204
Mining of other metals,Manufacture of wearing apparel,23.30335927
Mining of other metals,Manufacture of leather products,0.007941965
Mining of other metals,Manufacture of footwear,0.146158974
Mining of other metals,Sawmilling and planing of wood,19.03859859
Mining of other metals,Manufacture of wood products,8.260436778
Mining of other metals,Manufacture of pulp and paper,14.69708075
Mining of other metals,Manufacture of containers of paper,2.486270601
Mining of other metals,Manufacture of other paper products,17.7392889
Mining of other metals,Printing,10.45209128
Mining of other metals,Manufacture of refined petroleum products,20.28084079
Mining of other metals,Manufacture of basic chemicals,77089.60295
Mining of other metals,Manufacture of paints,5.33301708
Mining of other metals,Manufacture of pharmaceutical products,61.90558104
Mining of other metals,Manufacture of soap detergents and toilet products,74.32443027
Mining of other metals,Manufacture of other chemical products,129.3824251
Mining of other metals,Manufacture of rubber products,0.771327596
Mining of other metals,Manufacture of plastic products,5.1554982
Mining of other metals,Manufacture of glass and glass products,366.1558887
Mining of other metals,Manufacture of cement,7502.171625
Mining of other metals,Manufacture of concrete and concrete products,15520.84845
Mining of other metals,Manufacture of basic iron and steel,324.5959745
Mining of other metals,Manufacture of other basic metals,19.98737336
Mining of other metals,Manufacture of fabricated metal products,265.0897889
Mining of other metals,Manufacture of industrial and domestic machinery and equipment,526.4228275
Mining of other metals,Manufacture of electric and electronic machinery and equipment,28.47096678
Mining of other metals,Manufacture of transport equipment,5.706249808
Mining of other metals,Manufacture of furniture,30.28168018
Mining of other metals,Repair and installation of machinery and other manufacturing,44.47739695
Mining of other metals,Electric power generation,372.7609346
Mining of other metals,Transmission of electric power,0.786889554
Mining of other metals,Distribution of electric power,14.72107045
Mining of other metals,Gas and steam manufacture and supply,2.033548592
Mining of other metals,Water collection treatment and supply,3.287845994
Mining of other metals,Waste collection and recycling activities,4.936238952
Mining of other metals,Construction of residential buildings,16.65803788
Mining of other metals,Construction of non-residential buildings,474.1874725
Mining of other metals,Civil engineering,4832.61703
Mining of other metals,Specialized construction activities,934.5519027
Mining of other metals,Wholesale and retail trade of motor vehicles,16.22655395
Mining of other metals,Wholesale trade,139.7112322
Mining of other metals,Retail trade,122.2448857
Mining of other metals,Accommodation,2.459782464
Mining of other metals,Food and beverage service activities,11.26880363
Mining of other metals,Transport via railways,1.585398247
Mining of other metals,Other passenger land transport,2.4783959
Mining of other metals,Freight transport by road,6.529126087
Mining of other metals,Transport via pipeline,0.208117686
Mining of other metals,Water transport,2.0668502
Mining of other metals,Air transport,40.2773961
Mining of other metals,Warehousing,0.402372409
Mining of other metals,Support activities for land transport,5.80228603
Mining of other metals,Other support activities for transport,28.63676452
Mining of other metals,Postal and courier activities,0
Mining of other metals,Wireless telecommunications,19.35299326
Mining of other metals,Wired telecommunications,62.47936559
Mining of other metals,Other telecommunications activities,10.70369049
Mining of other metals,Information services activities,314.8436071
Mining of other metals,Publishing activities,9.361353033
Mining of other metals,Banking monetary intermediation,259.4104542
Mining of other metals,Insurance activities,55.84559803
Mining of other metals,Auxiliary financial activities,51.62275868
Mining of other metals,Real estate activities,8.313151698
Mining of other metals,Housing services,0
Mining of other metals,Legal and accounting services,11.47896259
Mining of other metals,Architectural and engineering services,55.99013493
Mining of other metals,Other professional activities,169.5087586
Mining of other metals,Rental and leasing activities,18.84791134
Mining of other metals,Business support activities,98.09888166
Mining of other metals,Public administration,159.3828632
Mining of other metals,Public education,2.321709306
Mining of other metals,Private education,8.21349786
Mining of other metals,Public health activities,42.69212443
Mining of other metals,Private health activities,4.99620238
Mining of other metals,Activities of organizations,1.648558334
Mining of other metals,Artistic and entertainment activities,15.84176493
Mining of other metals,Other personal services,3.118328773
Other mining activities and mining services,Cultivation of annual crops,2074.357811
Other mining activities and mining services,Cultivation of vegetables,2153.535168
Other mining activities and mining services,Cultivation of grapes,9289.065215
Other mining activities and mining services,Cultivation of other fruits,29689.23945
Other mining activities and mining services,Cattle bredding,192.5558483
Other mining activities and mining services,Pigs breeding,12.39277448
Other mining activities and mining services,Chicken breeding,72.20882364
Other mining activities and mining services,Breeding of other animals,45.27422316
Other mining activities and mining services,Farming services,6872.075352
Other mining activities and mining services,Forestry,25.78269644
Other mining activities and mining services,Aquaculture,163.6707932
Other mining activities and mining services,Extractive fishing,3.156814145
Other mining activities and mining services,Coal mining,1.129737577
Other mining activities and mining services,Crude oil and gas mining,6.984596676
Other mining activities and mining services,Copper mining,4979.299171
Other mining activities and mining services,Iron mining,734.0917354
Other mining activities and mining services,Mining of other metals,92.17830655
Other mining activities and mining services,Other mining activities and mining services,707.7055824
Other mining activities and mining services,Processing and preserving of meat,122.0322256
Other mining activities and mining services,Processing and preserving of fishmeal and fish oil,3.311429092
Other mining activities and mining services,Processing and preserving of fish,2272.372142
Other mining activities and mining services,Processing and preserving of fruits and vegetables,90.90866007
Other mining activities and mining services,Manufacture of vegetable and animal oils,1.85350514
Other mining activities and mining services,Manufacture of dairy products,260.7211333
Other mining activities and mining services,Manufacture of grain mill products,64.00228334
Other mining activities and mining services,Manufacture of prepared animal feeds,297.2641537
Other mining activities and mining services,Manufacture of bakery products,41.02410334
Other mining activities and mining services,Manufacture of macaroni noodles and other related products,0.345536419
Other mining activities and mining services,Manufacture of other food products,976.0400136
Other mining activities and mining services,Distilling rectifying and blending of spirits,1.708744473
Other mining activities and mining services,Manufacture of wines,31.46750243
Other mining activities and mining services,Manufacture of beers,9.105917856
Other mining activities and mining services,Manufacture of soft drinks,60.32034582
Other mining activities and mining services,Manufacture of tobacco products,1.542639252
Other mining activities and mining services,Manufacture of textile products,241.0806545
Other mining activities and mining services,Manufacture of wearing apparel,19.38699454
Other mining activities and mining services,Manufacture of leather products,46.10030427
Other mining activities and mining services,Manufacture of footwear,29.84419909
Other mining activities and mining services,Sawmilling and planing of wood,20.62085461
Other mining activities and mining services,Manufacture of wood products,341.0595326
Other mining activities and mining services,Manufacture of pulp and paper,636.9101961
Other mining activities and mining services,Manufacture of containers of paper,190.3778458
Other mining activities and mining services,Manufacture of other paper products,43.19402342
Other mining activities and mining services,Printing,14.74522435
Other mining activities and mining services,Manufacture of refined petroleum products,19.80085757
Other mining activities and mining services,Manufacture of basic chemicals,220841.0757
Other mining activities and mining services,Manufacture of paints,50.25476217
Other mining activities and mining services,Manufacture of pharmaceutical products,414.9332305
Other mining activities and mining services,Manufacture of soap detergents and toilet products,622.6962537
Other mining activities and mining services,Manufacture of other chemical products,2055.86507
Other mining activities and mining services,Manufacture of rubber products,146.2264951
Other mining activities and mining services,Manufacture of plastic products,898.0320094
Other mining activities and mining services,Manufacture of glass and glass products,3818.644946
Other mining activities and mining services,Manufacture of cement,77863.22283
Other mining activities and mining services,Manufacture of concrete and concrete products,161061.6135
Other mining activities and mining services,Manufacture of basic iron and steel,3365.790829
Other mining activities and mining services,Manufacture of other basic metals,165.3154314
Other mining activities and mining services,Manufacture of fabricated metal products,2397.552887
Other mining activities and mining services,Manufacture of industrial and domestic machinery and equipment,2998.606155
Other mining activities and mining services,Manufacture of electric and electronic machinery and equipment,219.8061168
Other mining activities and mining services,Manufacture of transport equipment,6.955897884
Other mining activities and mining services,Manufacture of furniture,24.35186249
Other mining activities and mining services,Repair and installation of machinery and other manufacturing,455.9662506
Other mining activities and mining services,Electric power generation,3854.351901
Other mining activities and mining services,Transmission of electric power,0.587089902
Other mining activities and mining services,Distribution of electric power,9.627331787
Other mining activities and mining services,Gas and steam manufacture and supply,12.2019375
Other mining activities and mining services,Water collection treatment and supply,89.31873475
Other mining activities and mining services,Waste collection and recycling activities,18.87890447
Other mining activities and mining services,Construction of residential buildings,198.980523
Other mining activities and mining services,Construction of non-residential buildings,4918.704521
Other mining activities and mining services,Civil engineering,50130.94321
Other mining activities and mining services,Specialized construction activities,9693.756146
Other mining activities and mining services,Wholesale and retail trade of motor vehicles,21.56757011
Other mining activities and mining services,Wholesale trade,188.0006902
Other mining activities and mining services,Retail trade,238.6714903
Other mining activities and mining services,Accommodation,5.492420183
Other mining activities and mining services,Food and beverage service activities,15.77061751
Other mining activities and mining services,Transport via railways,1.623071784
Other mining activities and mining services,Other passenger land transport,3.431391629
Other mining activities and mining services,Freight transport by road,16.65515636
Other mining activities and mining services,Transport via pipeline,0.156794191
Other mining activities and mining services,Water transport,1.622809665
Other mining activities and mining services,Air transport,30.46342792
Other mining activities and mining services,Warehousing,2.979900381
Other mining activities and mining services,Support activities for land transport,6.721154274
Other mining activities and mining services,Other support activities for transport,19.67240195
Other mining activities and mining services,Postal and courier activities,0.293644541
Other mining activities and mining services,Wireless telecommunications,23.63192214
Other mining activities and mining services,Wired telecommunications,41.32842285
Other mining activities and mining services,Other telecommunications activities,16.18560983
Other mining activities and mining services,Information services activities,209.5412004
Other mining activities and mining services,Publishing activities,13.85080481
Other mining activities and mining services,Banking monetary intermediation,187.5871864
Other mining activities and mining services,Insurance activities,50.26276768
Other mining activities and mining services,Auxiliary financial activities,103.0477231
Other mining activities and mining services,Real estate activities,25.88511052
Other mining activities and mining services,Housing services,0
Other mining activities and mining services,Legal and accounting services,18.99821403
Other mining activities and mining services,Architectural and engineering services,61.29610355
Other mining activities and mining services,Other professional activities,152.5622043
Other mining activities and mining services,Rental and leasing activities,19.59145191
Other mining activities and mining services,Business support activities,97.86991659
Other mining activities and mining services,Public administration,368.669464
Other mining activities and mining services,Public education,17.61246806
Other mining activities and mining services,Private education,47.51637629
Other mining activities and mining services,Public health activities,45.40967959
Other mining activities and mining services,Private health activities,130.5378889
Other mining activities and mining services,Activities of organizations,21.97663612
Other mining activities and mining services,Artistic and entertainment activities,15.74368067
Other mining activities and mining services,Other personal services,3.182812917
Processing and preserving of meat,Cultivation of annual crops,316.3624502
Processing and preserving of meat,Cultivation of vegetables,76.97217226
Processing and preserving of meat,Cultivation of grapes,159.9695221
Processing and preserving of meat,Cultivation of other fruits,391.1799306
Processing and preserving of meat,Cattle bredding,256.8824957
Processing and preserving of meat,Pigs breeding,118.3007668
Processing and preserving of meat,Chicken breeding,120.5109197
Processing and preserving of meat,Breeding of other animals,29.52584408
Processing and preserving of meat,Farming services,201.0080016
Processing and preserving of meat,Forestry,73.08050464
Processing and preserving of meat,Aquaculture,125.012932
Processing and preserving of meat,Extractive fishing,2242.244281
Processing and preserving of meat,Coal mining,8.411756814
Processing and preserving of meat,Crude oil and gas mining,3.27541582
Processing and preserving of meat,Copper mining,1861.702626
Processing and preserving of meat,Iron mining,26.37224612
Processing and preserving of meat,Mining of other metals,48.4322884
Processing and preserving of meat,Other mining activities and mining services,39.49968058
Processing and preserving of meat,Processing and preserving of meat,169723.7361
Processing and preserving of meat,Processing and preserving of fishmeal and fish oil,45.75964363
Processing and preserving of meat,Processing and preserving of fish,539.8947894
Processing and preserving of meat,Processing and preserving of fruits and vegetables,338.1390255
Processing and preserving of meat,Manufacture of vegetable and animal oils,448.539208
Processing and preserving of meat,Manufacture of dairy products,498.7117875
Processing and preserving of meat,Manufacture of grain mill products,280.1298381
Processing and preserving of meat,Manufacture of prepared animal feeds,42542.25285
Processing and preserving of meat,Manufacture of bakery products,752.5900114
Processing and preserving of meat,Manufacture of macaroni noodles and other related products,370.7223348
Processing and preserving of meat,Manufacture of other food products,6468.716973
Processing and preserving of meat,Distilling rectifying and blending of spirits,111.8210274
Processing and preserving of meat,Manufacture of wines,792.0052369
Processing and preserving of meat,Manufacture of beers,371.7581346
Processing and preserving of meat,Manufacture of soft drinks,739.0181646
Processing and preserving of meat,Manufacture of tobacco products,92.78632949
Processing and preserving of meat,Manufacture of textile products,161.6432705
Processing and preserving of meat,Manufacture of wearing apparel,2837.438101
Processing and preserving of meat,Manufacture of leather products,7249.181807
Processing and preserving of meat,Manufacture of footwear,620.5181368
Processing and preserving of meat,Sawmilling and planing of wood,460.4264871
Processing and preserving of meat,Manufacture of wood products,454.7191745
Processing and preserving of meat,Manufacture of pulp and paper,515.4734761
Processing and preserving of meat,Manufacture of containers of paper,188.270339
Processing and preserving of meat,Manufacture of other paper products,387.2734653
Processing and preserving of meat,Printing,248.8326279
Processing and preserving of meat,Manufacture of refined petroleum products,317.5468337
Processing and preserving of meat,Manufacture of basic chemicals,388.5084121
Processing and preserving of meat,Manufacture of paints,147.2573056
Processing and preserving of meat,Manufacture of pharmaceutical products,994.4852833
Processing and preserving of meat,Manufacture of soap detergents and toilet products,533.5433643
Processing and preserving of meat,Manufacture of other chemical products,272.7308759
Processing and preserving of meat,Manufacture of rubber products,29.27711197
Processing and preserving of meat,Manufacture of plastic products,448.8818586
Processing and preserving of meat,Manufacture of glass and glass products,103.241177
Processing and preserving of meat,Manufacture of cement,235.8262116
Processing and preserving of meat,Manufacture of concrete and concrete products,667.5509516
Processing and preserving of meat,Manufacture of basic iron and steel,328.1107196
Processing and preserving of meat,Manufacture of other basic metals,150.3935846
Processing and preserving of meat,Manufacture of fabricated metal products,875.2325838
Processing and preserving of meat,Manufacture of industrial and domestic machinery and equipment,527.8938453
Processing and preserving of meat,Manufacture of electric and electronic machinery and equipment,184.274322
Processing and preserving of meat,Manufacture of transport equipment,240.1570177
Processing and preserving of meat,Manufacture of furniture,328.3089439
Processing and preserving of meat,Repair and installation of machinery and other manufacturing,289.7173047
Processing and preserving of meat,Electric power generation,350.7230593
Processing and preserving of meat,Transmission of electric power,2.957115697
Processing and preserving of meat,Distribution of electric power,25.28510402
Processing and preserving of meat,Gas and steam manufacture and supply,732.9046486
Processing and preserving of meat,Water collection treatment and supply,44.92148029
Processing and preserving of meat,Waste collection and recycling activities,37.55376587
Processing and preserving of meat,Construction of residential buildings,1172.053681
Processing and preserving of meat,Construction of non-residential buildings,572.5123462
Processing and preserving of meat,Civil engineering,845.8935388
Processing and preserving of meat,Specialized construction activities,900.6165861
Processing and preserving of meat,Wholesale and retail trade of motor vehicles,980.6736807
Processing and preserving of meat,Wholesale trade,5020.835918
Processing and preserving of meat,Retail trade,43847.67281
Processing and preserving of meat,Accommodation,21338.6046
Processing and preserving of meat,Food and beverage service activities,332571.8572
Processing and preserving of meat,Transport via railways,55.31199491
Processing and preserving of meat,Other passenger land transport,293.4378672
Processing and preserving of meat,Freight transport by road,454.2987591
Processing and preserving of meat,Transport via pipeline,0.206829103
Processing and preserving of meat,Water transport,65.86745932
Processing and preserving of meat,Air transport,309.8607495
Processing and preserving of meat,Warehousing,108.439698
Processing and preserving of meat,Support activities for land transport,179.1467575
Processing and preserving of meat,Other support activities for transport,170.3057126
Processing and preserving of meat,Postal and courier activities,17.55632688
Processing and preserving of meat,Wireless telecommunications,1098.683638
Processing and preserving of meat,Wired telecommunications,303.6202607
Processing and preserving of meat,Other telecommunications activities,834.8950203
Processing and preserving of meat,Information services activities,1039.008949
Processing and preserving of meat,Publishing activities,514.9719115
Processing and preserving of meat,Banking monetary intermediation,1784.290383
Processing and preserving of meat,Insurance activities,1049.215144
Processing and preserving of meat,Auxiliary financial activities,474.9793245
Processing and preserving of meat,Real estate activities,973.2164359
Processing and preserving of meat,Housing services,0
Processing and preserving of meat,Legal and accounting services,670.2041383
Processing and preserving of meat,Architectural and engineering services,1376.477059
Processing and preserving of meat,Other professional activities,2936.488808
Processing and preserving of meat,Rental and leasing activities,522.8027044
Processing and preserving of meat,Business support activities,2440.068808
Processing and preserving of meat,Public administration,21528.19763
Processing and preserving of meat,Public education,5775.847756
Processing and preserving of meat,Private education,6549.371677
Processing and preserving of meat,Public health activities,21695.25406
Processing and preserving of meat,Private health activities,8964.259405
Processing and preserving of meat,Activities of organizations,5376.750989
Processing and preserving of meat,Artistic and entertainment activities,328.1966008
Processing and preserving of meat,Other personal services,105.2853695
Processing and preserving of fishmeal and fish oil,Cultivation of annual crops,0.06032817
Processing and preserving of fishmeal and fish oil,Cultivation of vegetables,0.045753931
Processing and preserving of fishmeal and fish oil,Cultivation of grapes,0.085511307
Processing and preserving of fishmeal and fish oil,Cultivation of other fruits,0.162691617
Processing and preserving of fishmeal and fish oil,Cattle bredding,0.058072061
Processing and preserving of fishmeal and fish oil,Pigs breeding,0.008644277
Processing and preserving of fishmeal and fish oil,Chicken breeding,0.010598965
Processing and preserving of fishmeal and fish oil,Breeding of other animals,0.004071837
Processing and preserving of fishmeal and fish oil,Farming services,0.031327346
Processing and preserving of fishmeal and fish oil,Forestry,0.079120088
Processing and preserving of fishmeal and fish oil,Aquaculture,110.6763957
Processing and preserving of fishmeal and fish oil,Extractive fishing,0.015722372
Processing and preserving of fishmeal and fish oil,Coal mining,0.02422202
Processing and preserving of fishmeal and fish oil,Crude oil and gas mining,0.069680883
Processing and preserving of fishmeal and fish oil,Copper mining,301.9722964
Processing and preserving of fishmeal and fish oil,Iron mining,8.629771585
Processing and preserving of fishmeal and fish oil,Mining of other metals,5.59010712
Processing and preserving of fishmeal and fish oil,Other mining activities and mining services,5.393301016
Processing and preserving of fishmeal and fish oil,Processing and preserving of meat,637.0375892
Processing and preserving of fishmeal and fish oil,Processing and preserving of fishmeal and fish oil,18521.04882
Processing and preserving of fishmeal and fish oil,Processing and preserving of fish,5270.430546
Processing and preserving of fishmeal and fish oil,Processing and preserving of fruits and vegetables,2.494512573
Processing and preserving of fishmeal and fish oil,Manufacture of vegetable and animal oils,0.02567266
Processing and preserving of fishmeal and fish oil,Manufacture of dairy products,6.99584835
Processing and preserving of fishmeal and fish oil,Manufacture of grain mill products,2.419334895
Processing and preserving of fishmeal and fish oil,Manufacture of prepared animal feeds,370450.6699
Processing and preserving of fishmeal and fish oil,Manufacture of bakery products,107.1496866
Processing and preserving of fishmeal and fish oil,Manufacture of macaroni noodles and other related products,0.205491097
Processing and preserving of fishmeal and fish oil,Manufacture of other food products,6.06604684
Processing and preserving of fishmeal and fish oil,Distilling rectifying and blending of spirits,0.029716791
Processing and preserving of fishmeal and fish oil,Manufacture of wines,3.439693023
Processing and preserving of fishmeal and fish oil,Manufacture of beers,0.664741822
Processing and preserving of fishmeal and fish oil,Manufacture of soft drinks,4.761786227
Processing and preserving of fishmeal and fish oil,Manufacture of tobacco products,0.001769387
Processing and preserving of fishmeal and fish oil,Manufacture of textile products,1.002822549
Processing and preserving of fishmeal and fish oil,Manufacture of wearing apparel,2.978530109
Processing and preserving of fishmeal and fish oil,Manufacture of leather products,0.001011808
Processing and preserving of fishmeal and fish oil,Manufacture of footwear,0.01862069
Processing and preserving of fishmeal and fish oil,Sawmilling and planing of wood,9.516068997
Processing and preserving of fishmeal and fish oil,Manufacture of wood products,9.34396235
Processing and preserving of fishmeal and fish oil,Manufacture of pulp and paper,64.53591123
Processing and preserving of fishmeal and fish oil,Manufacture of containers of paper,1.178341621
Processing and preserving of fishmeal and fish oil,Manufacture of other paper products,5.952661068
Processing and preserving of fishmeal and fish oil,Printing,1.332066014
Processing and preserving of fishmeal and fish oil,Manufacture of refined petroleum products,10.38960286
Processing and preserving of fishmeal and fish oil,Manufacture of basic chemicals,18.63406608
Processing and preserving of fishmeal and fish oil,Manufacture of paints,0.679427713
Processing and preserving of fishmeal and fish oil,Manufacture of pharmaceutical products,4.200757949
Processing and preserving of fishmeal and fish oil,Manufacture of soap detergents and toilet products,3.18652983
Processing and preserving of fishmeal and fish oil,Manufacture of other chemical products,0.181701947
Processing and preserving of fishmeal and fish oil,Manufacture of rubber products,0.531784837
Processing and preserving of fishmeal and fish oil,Manufacture of plastic products,8.794107144
Processing and preserving of fishmeal and fish oil,Manufacture of glass and glass products,2.9765077
Processing and preserving of fishmeal and fish oil,Manufacture of cement,7.298882403
Processing and preserving of fishmeal and fish oil,Manufacture of concrete and concrete products,121.6299146
Processing and preserving of fishmeal and fish oil,Manufacture of basic iron and steel,4.118534145
Processing and preserving of fishmeal and fish oil,Manufacture of other basic metals,1.034410926
Processing and preserving of fishmeal and fish oil,Manufacture of fabricated metal products,13.43095085
Processing and preserving of fishmeal and fish oil,Manufacture of industrial and domestic machinery and equipment,2.269833808
Processing and preserving of fishmeal and fish oil,Manufacture of electric and electronic machinery and equipment,1.082623813
Processing and preserving of fishmeal and fish oil,Manufacture of transport equipment,1.616275728
Processing and preserving of fishmeal and fish oil,Manufacture of furniture,4.957138881
Processing and preserving of fishmeal and fish oil,Repair and installation of machinery and other manufacturing,11.84868312
Processing and preserving of fishmeal and fish oil,Electric power generation,0.257033322
Processing and preserving of fishmeal and fish oil,Transmission of electric power,0.100249926
Processing and preserving of fishmeal and fish oil,Distribution of electric power,443.390604
Processing and preserving of fishmeal and fish oil,Gas and steam manufacture and supply,2.616275526
Processing and preserving of fishmeal and fish oil,Water collection treatment and supply,0.418872404
Processing and preserving of fishmeal and fish oil,Waste collection and recycling activities,0.62887808
Processing and preserving of fishmeal and fish oil,Construction of residential buildings,0.341826139
Processing and preserving of fishmeal and fish oil,Construction of non-residential buildings,0.111380533
Processing and preserving of fishmeal and fish oil,Civil engineering,1.341922688
Processing and preserving of fishmeal and fish oil,Specialized construction activities,0.224671996
Processing and preserving of fishmeal and fish oil,Wholesale and retail trade of motor vehicles,2.067267042
Processing and preserving of fishmeal and fish oil,Wholesale trade,29012.56397
Processing and preserving of fishmeal and fish oil,Retail trade,15.57402909
Processing and preserving of fishmeal and fish oil,Accommodation,0.313376903
Processing and preserving of fishmeal and fish oil,Food and beverage service activities,1619.003531
Processing and preserving of fishmeal and fish oil,Transport via railways,0.201980134
Processing and preserving of fishmeal and fish oil,Other passenger land transport,0.315748259
Processing and preserving of fishmeal and fish oil,Freight transport by road,0.719950212
Processing and preserving of fishmeal and fish oil,Transport via pipeline,0.026514245
Processing and preserving of fishmeal and fish oil,Water transport,0.263317234
Processing and preserving of fishmeal and fish oil,Air transport,5.131350363
Processing and preserving of fishmeal and fish oil,Warehousing,0.051262346
Processing and preserving of fishmeal and fish oil,Support activities for land transport,0.739212695
Processing and preserving of fishmeal and fish oil,Other support activities for transport,3.64833098
Processing and preserving of fishmeal and fish oil,Postal and courier activities,0
Processing and preserving of fishmeal and fish oil,Wireless telecommunications,2.465576194
Processing and preserving of fishmeal and fish oil,Wired telecommunications,7.959886843
Processing and preserving of fishmeal and fish oil,Other telecommunications activities,1.363652853
Processing and preserving of fishmeal and fish oil,Information services activities,40.11115449
Processing and preserving of fishmeal and fish oil,Publishing activities,1.192638724
Processing and preserving of fishmeal and fish oil,Banking monetary intermediation,33.04895692
Processing and preserving of fishmeal and fish oil,Insurance activities,7.114743192
Processing and preserving of fishmeal and fish oil,Auxiliary financial activities,6.576752401
Processing and preserving of fishmeal and fish oil,Real estate activities,1.059097611
Processing and preserving of fishmeal and fish oil,Housing services,0
Processing and preserving of fishmeal and fish oil,Legal and accounting services,1.462422712
Processing and preserving of fishmeal and fish oil,Architectural and engineering services,7.133157229
Processing and preserving of fishmeal and fish oil,Other professional activities,21.59545836
Processing and preserving of fishmeal and fish oil,Rental and leasing activities,2.401228631
Processing and preserving of fishmeal and fish oil,Business support activities,12.49782212
Processing and preserving of fishmeal and fish oil,Public administration,17.47700865
Processing and preserving of fishmeal and fish oil,Public education,0.295786348
Processing and preserving of fishmeal and fish oil,Private education,1.009100492
Processing and preserving of fishmeal and fish oil,Public health activities,5.438987357
Processing and preserving of fishmeal and fish oil,Private health activities,0.636517436
Processing and preserving of fishmeal and fish oil,Activities of organizations,0.210026745
Processing and preserving of fishmeal and fish oil,Artistic and entertainment activities,2.018244824
Processing and preserving of fishmeal and fish oil,Other personal services,0.397275868
Processing and preserving of fish,Cultivation of annual crops,137.7847333
Processing and preserving of fish,Cultivation of vegetables,83.87114514
Processing and preserving of fish,Cultivation of grapes,137.2869629
Processing and preserving of fish,Cultivation of other fruits,171.0071897
Processing and preserving of fish,Cattle bredding,62.92928998
Processing and preserving of fish,Pigs breeding,17.71975891
Processing and preserving of fish,Chicken breeding,31.30631039
Processing and preserving of fish,Breeding of other animals,9.374642283
Processing and preserving of fish,Farming services,19.98973463
Processing and preserving of fish,Forestry,102.3861492
Processing and preserving of fish,Aquaculture,2250.793015
Processing and preserving of fish,Extractive fishing,2457.99346
Processing and preserving of fish,Coal mining,385.0852997
Processing and preserving of fish,Crude oil and gas mining,4.195289536
Processing and preserving of fish,Copper mining,23478.25183
Processing and preserving of fish,Iron mining,96.45912759
Processing and preserving of fish,Mining of other metals,395.9606485
Processing and preserving of fish,Other mining activities and mining services,90.44298989
Processing and preserving of fish,Processing and preserving of meat,1023.954392
Processing and preserving of fish,Processing and preserving of fishmeal and fish oil,65182.87264
Processing and preserving of fish,Processing and preserving of fish,57249.78388
Processing and preserving of fish,Processing and preserving of fruits and vegetables,671.876976
Processing and preserving of fish,Manufacture of vegetable and animal oils,335.8229278
Processing and preserving of fish,Manufacture of dairy products,638.9906806
Processing and preserving of fish,Manufacture of grain mill products,404.2425819
Processing and preserving of fish,Manufacture of prepared animal feeds,3692.854559
Processing and preserving of fish,Manufacture of bakery products,932.3320106
Processing and preserving of fish,Manufacture of macaroni noodles and other related products,5.914387993
Processing and preserving of fish,Manufacture of other food products,2934.270248
Processing and preserving of fish,Distilling rectifying and blending of spirits,101.7783707
Processing and preserving of fish,Manufacture of wines,2073.761226
Processing and preserving of fish,Manufacture of beers,405.0811735
Processing and preserving of fish,Manufacture of soft drinks,595.1847573
Processing and preserving of fish,Manufacture of tobacco products,76.92966623
Processing and preserving of fish,Manufacture of textile products,1339.911389
Processing and preserving of fish,Manufacture of wearing apparel,296.6870961
Processing and preserving of fish,Manufacture of leather products,249.5769504
Processing and preserving of fish,Manufacture of footwear,114.2625244
Processing and preserving of fish,Sawmilling and planing of wood,911.6558398
Processing and preserving of fish,Manufacture of wood products,2525.645231
Processing and preserving of fish,Manufacture of pulp and paper,1480.747388
Processing and preserving of fish,Manufacture of containers of paper,536.4086363
Processing and preserving of fish,Manufacture of other paper products,1033.611453
Processing and preserving of fish,Printing,269.3891928
Processing and preserving of fish,Manufacture of refined petroleum products,1295.047513
Processing and preserving of fish,Manufacture of basic chemicals,564.9008441
Processing and preserving of fish,Manufacture of paints,456.651184
Processing and preserving of fish,Manufacture of pharmaceutical products,870.0440974
Processing and preserving of fish,Manufacture of soap detergents and toilet products,4206.955546
Processing and preserving of fish,Manufacture of other chemical products,1794.699634
Processing and preserving of fish,Manufacture of rubber products,54.57471023
Processing and preserving of fish,Manufacture of plastic products,485.3491915
Processing and preserving of fish,Manufacture of glass and glass products,73.11897921
Processing and preserving of fish,Manufacture of cement,201.6628403
Processing and preserving of fish,Manufacture of concrete and concrete products,3389.582522
Processing and preserving of fish,Manufacture of basic iron and steel,225.1956493
Processing and preserving of fish,Manufacture of other basic metals,84.31037579
Processing and preserving of fish,Manufacture of fabricated metal products,652.7561987
Processing and preserving of fish,Manufacture of industrial and domestic machinery and equipment,660.3699822
Processing and preserving of fish,Manufacture of electric and electronic machinery and equipment,266.2916375
Processing and preserving of fish,Manufacture of transport equipment,170.8908078
Processing and preserving of fish,Manufacture of furniture,1168.870987
Processing and preserving of fish,Repair and installation of machinery and other manufacturing,187.7438438
Processing and preserving of fish,Electric power generation,246.6582823
Processing and preserving of fish,Transmission of electric power,3.19611331
Processing and preserving of fish,Distribution of electric power,62.37373164
Processing and preserving of fish,Gas and steam manufacture and supply,626.9325333
Processing and preserving of fish,Water collection treatment and supply,70.14119681
Processing and preserving of fish,Waste collection and recycling activities,32.70722936
Processing and preserving of fish,Construction of residential buildings,890.5233118
Processing and preserving of fish,Construction of non-residential buildings,245.9220408
Processing and preserving of fish,Civil engineering,6371.02993
Processing and preserving of fish,Specialized construction activities,2843.752385
Processing and preserving of fish,Wholesale and retail trade of motor vehicles,556.5396754
Processing and preserving of fish,Wholesale trade,5611.07578
Processing and preserving of fish,Retail trade,3523.217434
Processing and preserving of fish,Accommodation,12397.74233
Processing and preserving of fish,Food and beverage service activities,59501.42529
Processing and preserving of fish,Transport via railways,37.47452177
Processing and preserving of fish,Other passenger land transport,96.95077264
Processing and preserving of fish,Freight transport by road,129.6383059
Processing and preserving of fish,Transport via pipeline,0.364561031
Processing and preserving of fish,Water transport,23.19224672
Processing and preserving of fish,Air transport,257.5978248
Processing and preserving of fish,Warehousing,90.65494496
Processing and preserving of fish,Support activities for land transport,157.8169902
Processing and preserving of fish,Other support activities for transport,87.80657945
Processing and preserving of fish,Postal and courier activities,11.80435155
Processing and preserving of fish,Wireless telecommunications,497.7635512
Processing and preserving of fish,Wired telecommunications,210.2781781
Processing and preserving of fish,Other telecommunications activities,567.4413483
Processing and preserving of fish,Information services activities,1170.427298
Processing and preserving of fish,Publishing activities,393.2952776
Processing and preserving of fish,Banking monetary intermediation,1749.452558
Processing and preserving of fish,Insurance activities,939.0841739
Processing and preserving of fish,Auxiliary financial activities,457.1420157
Processing and preserving of fish,Real estate activities,799.9438613
Processing and preserving of fish,Housing services,0
Processing and preserving of fish,Legal and accounting services,581.6475219
Processing and preserving of fish,Architectural and engineering services,1962.221066
Processing and preserving of fish,Other professional activities,2635.437732
Processing and preserving of fish,Rental and leasing activities,394.9741684
Processing and preserving of fish,Business support activities,1927.387884
Processing and preserving of fish,Public administration,1919.966029
Processing and preserving of fish,Public education,1744.332878
Processing and preserving of fish,Private education,1428.588759
Processing and preserving of fish,Public health activities,3413.011741
Processing and preserving of fish,Private health activities,2093.018578
Processing and preserving of fish,Activities of organizations,2095.533498
Processing and preserving of fish,Artistic and entertainment activities,296.4225538
Processing and preserving of fish,Other personal services,26.72630442
Processing and preserving of fruits and vegetables,Cultivation of annual crops,211.2039839
Processing and preserving of fruits and vegetables,Cultivation of vegetables,54.98446217
Processing and preserving of fruits and vegetables,Cultivation of grapes,133.8991337
Processing and preserving of fruits and vegetables,Cultivation of other fruits,281.7604473
Processing and preserving of fruits and vegetables,Cattle bredding,177.0672359
Processing and preserving of fruits and vegetables,Pigs breeding,28.41877064
Processing and preserving of fruits and vegetables,Chicken breeding,36.76466062
Processing and preserving of fruits and vegetables,Breeding of other animals,22.25703278
Processing and preserving of fruits and vegetables,Farming services,119.3276273
Processing and preserving of fruits and vegetables,Forestry,96.27724489
Processing and preserving of fruits and vegetables,Aquaculture,89.23107642
Processing and preserving of fruits and vegetables,Extractive fishing,117.058859
Processing and preserving of fruits and vegetables,Coal mining,9.004449646
Processing and preserving of fruits and vegetables,Crude oil and gas mining,2.621233459
Processing and preserving of fruits and vegetables,Copper mining,1281.108507
Processing and preserving of fruits and vegetables,Iron mining,27.85979666
Processing and preserving of fruits and vegetables,Mining of other metals,43.48727608
Processing and preserving of fruits and vegetables,Other mining activities and mining services,26.10785675
Processing and preserving of fruits and vegetables,Processing and preserving of meat,529.7357078
Processing and preserving of fruits and vegetables,Processing and preserving of fishmeal and fish oil,18.9080485
Processing and preserving of fruits and vegetables,Processing and preserving of fish,183.5674038
Processing and preserving of fruits and vegetables,Processing and preserving of fruits and vegetables,331.9146364
Processing and preserving of fruits and vegetables,Manufacture of vegetable and animal oils,516.3021272
Processing and preserving of fruits and vegetables,Manufacture of dairy products,715.8032415
Processing and preserving of fruits and vegetables,Manufacture of grain mill products,1898.633539
Processing and preserving of fruits and vegetables,Manufacture of prepared animal feeds,669.9867806
Processing and preserving of fruits and vegetables,Manufacture of bakery products,6029.043356
Processing and preserving of fruits and vegetables,Manufacture of macaroni noodles and other related products,210.1393747
Processing and preserving of fruits and vegetables,Manufacture of other food products,8889.749087
Processing and preserving of fruits and vegetables,Distilling rectifying and blending of spirits,120.4806513
Processing and preserving of fruits and vegetables,Manufacture of wines,1053.149091
Processing and preserving of fruits and vegetables,Manufacture of beers,351.6020963
Processing and preserving of fruits and vegetables,Manufacture of soft drinks,7342.232616
Processing and preserving of fruits and vegetables,Manufacture of tobacco products,157.9609363
Processing and preserving of fruits and vegetables,Manufacture of textile products,81.08788167
Processing and preserving of fruits and vegetables,Manufacture of wearing apparel,198.2527078
Processing and preserving of fruits and vegetables,Manufacture of leather products,6.90494687
Processing and preserving of fruits and vegetables,Manufacture of footwear,18.5929058
Processing and preserving of fruits and vegetables,Sawmilling and planing of wood,137.6912319
Processing and preserving of fruits and vegetables,Manufacture of wood products,157.8220786
Processing and preserving of fruits and vegetables,Manufacture of pulp and paper,245.8198121
Processing and preserving of fruits and vegetables,Manufacture of containers of paper,71.88958234
Processing and preserving of fruits and vegetables,Manufacture of other paper products,164.9529392
Processing and preserving of fruits and vegetables,Printing,85.86182876
Processing and preserving of fruits and vegetables,Manufacture of refined petroleum products,113.0447165
Processing and preserving of fruits and vegetables,Manufacture of basic chemicals,189.3346627
Processing and preserving of fruits and vegetables,Manufacture of paints,60.52389736
Processing and preserving of fruits and vegetables,Manufacture of pharmaceutical products,361.364922
Processing and preserving of fruits and vegetables,Manufacture of soap detergents and toilet products,1695.599322
Processing and preserving of fruits and vegetables,Manufacture of other chemical products,195.7751771
Processing and preserving of fruits and vegetables,Manufacture of rubber products,21.47898861
Processing and preserving of fruits and vegetables,Manufacture of plastic products,249.7380832
Processing and preserving of fruits and vegetables,Manufacture of glass and glass products,59.07568368
Processing and preserving of fruits and vegetables,Manufacture of cement,90.2687318
Processing and preserving of fruits and vegetables,Manufacture of concrete and concrete products,274.574663
Processing and preserving of fruits and vegetables,Manufacture of basic iron and steel,197.1646304
Processing and preserving of fruits and vegetables,Manufacture of other basic metals,92.04623039
Processing and preserving of fruits and vegetables,Manufacture of fabricated metal products,375.3513333
Processing and preserving of fruits and vegetables,Manufacture of industrial and domestic machinery and equipment,259.8256249
Processing and preserving of fruits and vegetables,Manufacture of electric and electronic machinery and equipment,96.86640668
Processing and preserving of fruits and vegetables,Manufacture of transport equipment,92.56694189
Processing and preserving of fruits and vegetables,Manufacture of furniture,189.1711656
Processing and preserving of fruits and vegetables,Repair and installation of machinery and other manufacturing,99.72973172
Processing and preserving of fruits and vegetables,Electric power generation,235.5093563
Processing and preserving of fruits and vegetables,Transmission of electric power,1.255982807
Processing and preserving of fruits and vegetables,Distribution of electric power,344.4552232
Processing and preserving of fruits and vegetables,Gas and steam manufacture and supply,188.1705464
Processing and preserving of fruits and vegetables,Water collection treatment and supply,15.77720472
Processing and preserving of fruits and vegetables,Waste collection and recycling activities,17.75832007
Processing and preserving of fruits and vegetables,Construction of residential buildings,547.1270852
Processing and preserving of fruits and vegetables,Construction of non-residential buildings,330.7797326
Processing and preserving of fruits and vegetables,Civil engineering,828.9692621
Processing and preserving of fruits and vegetables,Specialized construction activities,758.3995213
Processing and preserving of fruits and vegetables,Wholesale and retail trade of motor vehicles,486.4513786
Processing and preserving of fruits and vegetables,Wholesale trade,2573.827796
Processing and preserving of fruits and vegetables,Retail trade,7808.569233
Processing and preserving of fruits and vegetables,Accommodation,2667.20537
Processing and preserving of fruits and vegetables,Food and beverage service activities,27439.39937
Processing and preserving of fruits and vegetables,Transport via railways,75.33994803
Processing and preserving of fruits and vegetables,Other passenger land transport,211.8526159
Processing and preserving of fruits and vegetables,Freight transport by road,396.2063529
Processing and preserving of fruits and vegetables,Transport via pipeline,0.174891235
Processing and preserving of fruits and vegetables,Water transport,178.78745
Processing and preserving of fruits and vegetables,Air transport,148.2468399
Processing and preserving of fruits and vegetables,Warehousing,1103.743607
Processing and preserving of fruits and vegetables,Support activities for land transport,49.28241214
Processing and preserving of fruits and vegetables,Other support activities for transport,327.2356542
Processing and preserving of fruits and vegetables,Postal and courier activities,5.042686033
Processing and preserving of fruits and vegetables,Wireless telecommunications,615.259788
Processing and preserving of fruits and vegetables,Wired telecommunications,192.9096912
Processing and preserving of fruits and vegetables,Other telecommunications activities,331.8677431
Processing and preserving of fruits and vegetables,Information services activities,446.926535
Processing and preserving of fruits and vegetables,Publishing activities,174.0030752
Processing and preserving of fruits and vegetables,Banking monetary intermediation,605.9843845
Processing and preserving of fruits and vegetables,Insurance activities,301.7911624
Processing and preserving of fruits and vegetables,Auxiliary financial activities,161.6464142
Processing and preserving of fruits and vegetables,Real estate activities,277.5497469
Processing and preserving of fruits and vegetables,Housing services,0
Processing and preserving of fruits and vegetables,Legal and accounting services,177.6184807
Processing and preserving of fruits and vegetables,Architectural and engineering services,412.62509
Processing and preserving of fruits and vegetables,Other professional activities,874.9651095
Processing and preserving of fruits and vegetables,Rental and leasing activities,192.5863154
Processing and preserving of fruits and vegetables,Business support activities,852.7795916
Processing and preserving of fruits and vegetables,Public administration,2139.416277
Processing and preserving of fruits and vegetables,Public education,588.261377
Processing and preserving of fruits and vegetables,Private education,598.07244
Processing and preserving of fruits and vegetables,Public health activities,2449.227596
Processing and preserving of fruits and vegetables,Private health activities,1641.559879
Processing and preserving of fruits and vegetables,Activities of organizations,1529.787205
Processing and preserving of fruits and vegetables,Artistic and entertainment activities,132.3997993
Processing and preserving of fruits and vegetables,Other personal services,75.28850105
Manufacture of vegetable and animal oils,Cultivation of annual crops,1293.665052
Manufacture of vegetable and animal oils,Cultivation of vegetables,20.90286805
Manufacture of vegetable and animal oils,Cultivation of grapes,45.72751358
Manufacture of vegetable and animal oils,Cultivation of other fruits,112.4303724
Manufacture of vegetable and animal oils,Cattle bredding,2594.391707
Manufacture of vegetable and animal oils,Pigs breeding,3585.051234
Manufacture of vegetable and animal oils,Chicken breeding,2787.133563
Manufacture of vegetable and animal oils,Breeding of other animals,460.1189791
Manufacture of vegetable and animal oils,Farming services,30.61449909
Manufacture of vegetable and animal oils,Forestry,24.22962454
Manufacture of vegetable and animal oils,Aquaculture,34.52122617
Manufacture of vegetable and animal oils,Extractive fishing,31.36830212
Manufacture of vegetable and animal oils,Coal mining,2.551321243
Manufacture of vegetable and animal oils,Crude oil and gas mining,0.732756976
Manufacture of vegetable and animal oils,Copper mining,270.1203862
Manufacture of vegetable and animal oils,Iron mining,8.414506665
Manufacture of vegetable and animal oils,Mining of other metals,14.76114479
Manufacture of vegetable and animal oils,Other mining activities and mining services,8.027398067
Manufacture of vegetable and animal oils,Processing and preserving of meat,3808.239041
Manufacture of vegetable and animal oils,Processing and preserving of fishmeal and fish oil,4.903848062
Manufacture of vegetable and animal oils,Processing and preserving of fish,28.42732546
Manufacture of vegetable and animal oils,Processing and preserving of fruits and vegetables,261.5843785
Manufacture of vegetable and animal oils,Manufacture of vegetable and animal oils,16796.67471
Manufacture of vegetable and animal oils,Manufacture of dairy products,361.1640617
Manufacture of vegetable and animal oils,Manufacture of grain mill products,1482.664752
Manufacture of vegetable and animal oils,Manufacture of prepared animal feeds,1619.41043
Manufacture of vegetable and animal oils,Manufacture of bakery products,8861.125698
Manufacture of vegetable and animal oils,Manufacture of macaroni noodles and other related products,322.7246457
Manufacture of vegetable and animal oils,Manufacture of other food products,5557.218472
Manufacture of vegetable and animal oils,Distilling rectifying and blending of spirits,7.613834799
Manufacture of vegetable and animal oils,Manufacture of wines,43.64619559
Manufacture of vegetable and animal oils,Manufacture of beers,155.263356
Manufacture of vegetable and animal oils,Manufacture of soft drinks,89.23439312
Manufacture of vegetable and animal oils,Manufacture of tobacco products,7.533538693
Manufacture of vegetable and animal oils,Manufacture of textile products,20.27084106
Manufacture of vegetable and animal oils,Manufacture of wearing apparel,57.25783099
Manufacture of vegetable and animal oils,Manufacture of leather products,11.1496692
Manufacture of vegetable and animal oils,Manufacture of footwear,6.126129499
Manufacture of vegetable and animal oils,Sawmilling and planing of wood,16.06269801
Manufacture of vegetable and animal oils,Manufacture of wood products,23.91562678
Manufacture of vegetable and animal oils,Manufacture of pulp and paper,46.38969032
Manufacture of vegetable and animal oils,Manufacture of containers of paper,16.47947605
Manufacture of vegetable and animal oils,Manufacture of other paper products,37.96253759
Manufacture of vegetable and animal oils,Printing,16.94881464
Manufacture of vegetable and animal oils,Manufacture of refined petroleum products,15.73450828
Manufacture of vegetable and animal oils,Manufacture of basic chemicals,36.84378683
Manufacture of vegetable and animal oils,Manufacture of paints,14.43578214
Manufacture of vegetable and animal oils,Manufacture of pharmaceutical products,72.33144132
Manufacture of vegetable and animal oils,Manufacture of soap detergents and toilet products,4417.478121
Manufacture of vegetable and animal oils,Manufacture of other chemical products,508.7304089
Manufacture of vegetable and animal oils,Manufacture of rubber products,8.166959206
Manufacture of vegetable and animal oils,Manufacture of plastic products,83.05969869
Manufacture of vegetable and animal oils,Manufacture of glass and glass products,18.86030525
Manufacture of vegetable and animal oils,Manufacture of cement,19.89748843
Manufacture of vegetable and animal oils,Manufacture of concrete and concrete products,99.07164226
Manufacture of vegetable and animal oils,Manufacture of basic iron and steel,69.3762157
Manufacture of vegetable and animal oils,Manufacture of other basic metals,32.39542661
Manufacture of vegetable and animal oils,Manufacture of fabricated metal products,94.97762313
Manufacture of vegetable and animal oils,Manufacture of industrial and domestic machinery and equipment,80.75368561
Manufacture of vegetable and animal oils,Manufacture of electric and electronic machinery and equipment,30.13180239
Manufacture of vegetable and animal oils,Manufacture of transport equipment,22.47424403
Manufacture of vegetable and animal oils,Manufacture of furniture,56.33200705
Manufacture of vegetable and animal oils,Repair and installation of machinery and other manufacturing,30.25106278
Manufacture of vegetable and animal oils,Electric power generation,36.83390508
Manufacture of vegetable and animal oils,Transmission of electric power,0.181229195
Manufacture of vegetable and animal oils,Distribution of electric power,1.629072204
Manufacture of vegetable and animal oils,Gas and steam manufacture and supply,20.74742405
Manufacture of vegetable and animal oils,Water collection treatment and supply,2.898908469
Manufacture of vegetable and animal oils,Waste collection and recycling activities,4.574202851
Manufacture of vegetable and animal oils,Construction of residential buildings,168.1534341
Manufacture of vegetable and animal oils,Construction of non-residential buildings,112.6573769
Manufacture of vegetable and animal oils,Civil engineering,278.544736
Manufacture of vegetable and animal oils,Specialized construction activities,288.6502034
Manufacture of vegetable and animal oils,Wholesale and retail trade of motor vehicles,140.9097793
Manufacture of vegetable and animal oils,Wholesale trade,464.5692821
Manufacture of vegetable and animal oils,Retail trade,4594.212781
Manufacture of vegetable and animal oils,Accommodation,509.5616068
Manufacture of vegetable and animal oils,Food and beverage service activities,4556.854343
Manufacture of vegetable and animal oils,Transport via railways,2.908674754
Manufacture of vegetable and animal oils,Other passenger land transport,67.92237828
Manufacture of vegetable and animal oils,Freight transport by road,93.84264075
Manufacture of vegetable and animal oils,Transport via pipeline,0.02556894
Manufacture of vegetable and animal oils,Water transport,7.806939877
Manufacture of vegetable and animal oils,Air transport,29.13365931
Manufacture of vegetable and animal oils,Warehousing,3.264929949
Manufacture of vegetable and animal oils,Support activities for land transport,5.772862419
Manufacture of vegetable and animal oils,Other support activities for transport,28.90086884
Manufacture of vegetable and animal oils,Postal and courier activities,1.037403802
Manufacture of vegetable and animal oils,Wireless telecommunications,210.404085
Manufacture of vegetable and animal oils,Wired telecommunications,55.18097589
Manufacture of vegetable and animal oils,Other telecommunications activities,84.67728625
Manufacture of vegetable and animal oils,Information services activities,58.51087665
Manufacture of vegetable and animal oils,Publishing activities,36.32430389
Manufacture of vegetable and animal oils,Banking monetary intermediation,76.06839392
Manufacture of vegetable and animal oils,Insurance activities,36.80699583
Manufacture of vegetable and animal oils,Auxiliary financial activities,24.34287306
Manufacture of vegetable and animal oils,Real estate activities,42.65807654
Manufacture of vegetable and animal oils,Housing services,0
Manufacture of vegetable and animal oils,Legal and accounting services,20.61379773
Manufacture of vegetable and animal oils,Architectural and engineering services,58.49950542
Manufacture of vegetable and animal oils,Other professional activities,116.4579654
Manufacture of vegetable and animal oils,Rental and leasing activities,43.44418362
Manufacture of vegetable and animal oils,Business support activities,175.3237361
Manufacture of vegetable and animal oils,Public administration,1009.402412
Manufacture of vegetable and animal oils,Public education,34.93076226
Manufacture of vegetable and animal oils,Private education,78.15649622
Manufacture of vegetable and animal oils,Public health activities,411.3619616
Manufacture of vegetable and animal oils,Private health activities,1213.589143
Manufacture of vegetable and animal oils,Activities of organizations,230.784524
Manufacture of vegetable and animal oils,Artistic and entertainment activities,31.68321601
Manufacture of vegetable and animal oils,Other personal services,28.81396147
Manufacture of dairy products,Cultivation of annual crops,379.326593
Manufacture of dairy products,Cultivation of vegetables,90.15887864
Manufacture of dairy products,Cultivation of grapes,178.3703178
Manufacture of dairy products,Cultivation of other fruits,426.6483507
Manufacture of dairy products,Cattle bredding,366.4212151
Manufacture of dairy products,Pigs breeding,189.1917492
Manufacture of dairy products,Chicken breeding,169.8592233
Manufacture of dairy products,Breeding of other animals,49.76440386
Manufacture of dairy products,Farming services,115.4850202
Manufacture of dairy products,Forestry,98.36421483
Manufacture of dairy products,Aquaculture,131.4305698
Manufacture of dairy products,Extractive fishing,1004.620137
Manufacture of dairy products,Coal mining,10.39749416
Manufacture of dairy products,Crude oil and gas mining,2.71276032
Manufacture of dairy products,Copper mining,1843.207561
Manufacture of dairy products,Iron mining,36.09320845
Manufacture of dairy products,Mining of other metals,63.40219821
Manufacture of dairy products,Other mining activities and mining services,38.15643143
Manufacture of dairy products,Processing and preserving of meat,578.4594301
Manufacture of dairy products,Processing and preserving of fishmeal and fish oil,22.27739801
Manufacture of dairy products,Processing and preserving of fish,413.7918778
Manufacture of dairy products,Processing and preserving of fruits and vegetables,278.7235233
Manufacture of dairy products,Manufacture of vegetable and animal oils,758.1047035
Manufacture of dairy products,Manufacture of dairy products,41655.41321
Manufacture of dairy products,Manufacture of grain mill products,553.0619798
Manufacture of dairy products,Manufacture of prepared animal feeds,3417.642569
Manufacture of dairy products,Manufacture of bakery products,15569.23282
Manufacture of dairy products,Manufacture of macaroni noodles and other related products,190.2406796
Manufacture of dairy products,Manufacture of other food products,32674.07537
Manufacture of dairy products,Distilling rectifying and blending of spirits,2355.934868
Manufacture of dairy products,Manufacture of wines,251.0739639
Manufacture of dairy products,Manufacture of beers,297.381806
Manufacture of dairy products,Manufacture of soft drinks,936.8834531
Manufacture of dairy products,Manufacture of tobacco products,34.4725243
Manufacture of dairy products,Manufacture of textile products,90.70667574
Manufacture of dairy products,Manufacture of wearing apparel,243.4028358
Manufacture of dairy products,Manufacture of leather products,7.426665873
Manufacture of dairy products,Manufacture of footwear,26.35427001
Manufacture of dairy products,Sawmilling and planing of wood,153.0568306
Manufacture of dairy products,Manufacture of wood products,138.4307393
Manufacture of dairy products,Manufacture of pulp and paper,250.7064919
Manufacture of dairy products,Manufacture of containers of paper,78.72070713
Manufacture of dairy products,Manufacture of other paper products,178.9449668
Manufacture of dairy products,Printing,87.35169621
Manufacture of dairy products,Manufacture of refined petroleum products,101.172649
Manufacture of dairy products,Manufacture of basic chemicals,290.4266777
Manufacture of dairy products,Manufacture of paints,105.8520296
Manufacture of dairy products,Manufacture of pharmaceutical products,5391.040842
Manufacture of dairy products,Manufacture of soap detergents and toilet products,787.8138501
Manufacture of dairy products,Manufacture of other chemical products,221.4999198
Manufacture of dairy products,Manufacture of rubber products,47.52746101
Manufacture of dairy products,Manufacture of plastic products,343.5775009
Manufacture of dairy products,Manufacture of glass and glass products,89.55231341
Manufacture of dairy products,Manufacture of cement,98.17186089
Manufacture of dairy products,Manufacture of concrete and concrete products,302.0369285
Manufacture of dairy products,Manufacture of basic iron and steel,276.7236895
Manufacture of dairy products,Manufacture of other basic metals,133.4708512
Manufacture of dairy products,Manufacture of fabricated metal products,501.3651516
Manufacture of dairy products,Manufacture of industrial and domestic machinery and equipment,342.6366726
Manufacture of dairy products,Manufacture of electric and electronic machinery and equipment,135.8424742
Manufacture of dairy products,Manufacture of transport equipment,116.4428689
Manufacture of dairy products,Manufacture of furniture,255.0635008
Manufacture of dairy products,Repair and installation of machinery and other manufacturing,155.5277176
Manufacture of dairy products,Electric power generation,163.652441
Manufacture of dairy products,Transmission of electric power,1.118825982
Manufacture of dairy products,Distribution of electric power,240.4335808
Manufacture of dairy products,Gas and steam manufacture and supply,175.0538243
Manufacture of dairy products,Water collection treatment and supply,15.54932641
Manufacture of dairy products,Waste collection and recycling activities,20.60865903
Manufacture of dairy products,Construction of residential buildings,967.2938041
Manufacture of dairy products,Construction of non-residential buildings,469.6711607
Manufacture of dairy products,Civil engineering,2253.446544
Manufacture of dairy products,Specialized construction activities,10155.06684
Manufacture of dairy products,Wholesale and retail trade of motor vehicles,594.3218536
Manufacture of dairy products,Wholesale trade,2149.159495
Manufacture of dairy products,Retail trade,4133.139412
Manufacture of dairy products,Accommodation,16610.42893
Manufacture of dairy products,Food and beverage service activities,177474.5919
Manufacture of dairy products,Transport via railways,15.44265396
Manufacture of dairy products,Other passenger land transport,266.5322913
Manufacture of dairy products,Freight transport by road,412.6245504
Manufacture of dairy products,Transport via pipeline,0.16354617
Manufacture of dairy products,Water transport,31.38947242
Manufacture of dairy products,Air transport,141.8048592
Manufacture of dairy products,Warehousing,42.15020842
Manufacture of dairy products,Support activities for land transport,39.96637331
Manufacture of dairy products,Other support activities for transport,136.7939505
Manufacture of dairy products,Postal and courier activities,5.164705901
Manufacture of dairy products,Wireless telecommunications,848.4863611
Manufacture of dairy products,Wired telecommunications,237.6247391
Manufacture of dairy products,Other telecommunications activities,382.8493815
Manufacture of dairy products,Information services activities,387.0643269
Manufacture of dairy products,Publishing activities,223.9015287
Manufacture of dairy products,Banking monetary intermediation,513.657996
Manufacture of dairy products,Insurance activities,251.0901639
Manufacture of dairy products,Auxiliary financial activities,159.8553342
Manufacture of dairy products,Real estate activities,259.7433093
Manufacture of dairy products,Housing services,0
Manufacture of dairy products,Legal and accounting services,144.2599293
Manufacture of dairy products,Architectural and engineering services,361.2944285
Manufacture of dairy products,Other professional activities,751.9800807
Manufacture of dairy products,Rental and leasing activities,214.3240649
Manufacture of dairy products,Business support activities,888.8923075
Manufacture of dairy products,Public administration,6053.48351
Manufacture of dairy products,Public education,1058.691115
Manufacture of dairy products,Private education,1432.583987
Manufacture of dairy products,Public health activities,5106.002011
Manufacture of dairy products,Private health activities,3096.540284
Manufacture of dairy products,Activities of organizations,4336.584408
Manufacture of dairy products,Artistic and entertainment activities,143.9192448
Manufacture of dairy products,Other personal services,111.4915343
Manufacture of grain mill products,Cultivation of annual crops,38.2976914
Manufacture of grain mill products,Cultivation of vegetables,10.07165508
Manufacture of grain mill products,Cultivation of grapes,21.14135194
Manufacture of grain mill products,Cultivation of other fruits,366.8071787
Manufacture of grain mill products,Cattle bredding,14958.47903
Manufacture of grain mill products,Pigs breeding,1264.656884
Manufacture of grain mill products,Chicken breeding,2148.808097
Manufacture of grain mill products,Breeding of other animals,3145.378318
Manufacture of grain mill products,Farming services,56.19028038
Manufacture of grain mill products,Forestry,18.5002938
Manufacture of grain mill products,Aquaculture,13.13954757
Manufacture of grain mill products,Extractive fishing,8.176231059
Manufacture of grain mill products,Coal mining,0.895186453
Manufacture of grain mill products,Crude oil and gas mining,0.469396297
Manufacture of grain mill products,Copper mining,891.1652137
Manufacture of grain mill products,Iron mining,23.15984411
Manufacture of grain mill products,Mining of other metals,18.24681671
Manufacture of grain mill products,Other mining activities and mining services,19.2771201
Manufacture of grain mill products,Processing and preserving of meat,128.9322988
Manufacture of grain mill products,Processing and preserving of fishmeal and fish oil,6.54298426
Manufacture of grain mill products,Processing and preserving of fish,68.02488585
Manufacture of grain mill products,Processing and preserving of fruits and vegetables,623.8756158
Manufacture of grain mill products,Manufacture of vegetable and animal oils,332.8299734
Manufacture of grain mill products,Manufacture of dairy products,3121.234887
Manufacture of grain mill products,Manufacture of grain mill products,1845.317394
Manufacture of grain mill products,Manufacture of prepared animal feeds,46045.44567
Manufacture of grain mill products,Manufacture of bakery products,177799.3268
Manufacture of grain mill products,Manufacture of macaroni noodles and other related products,44314.81279
Manufacture of grain mill products,Manufacture of other food products,7269.753782
Manufacture of grain mill products,Distilling rectifying and blending of spirits,14.54284208
Manufacture of grain mill products,Manufacture of wines,85.06776574
Manufacture of grain mill products,Manufacture of beers,18759.65659
Manufacture of grain mill products,Manufacture of soft drinks,177.5155787
Manufacture of grain mill products,Manufacture of tobacco products,15.10286721
Manufacture of grain mill products,Manufacture of textile products,17.22683568
Manufacture of grain mill products,Manufacture of wearing apparel,47.50212682
Manufacture of grain mill products,Manufacture of leather products,1.162523591
Manufacture of grain mill products,Manufacture of footwear,3.87492009
Manufacture of grain mill products,Sawmilling and planing of wood,65.90752122
Manufacture of grain mill products,Manufacture of wood products,66.32899195
Manufacture of grain mill products,Manufacture of pulp and paper,205.0697227
Manufacture of grain mill products,Manufacture of containers of paper,20.54697445
Manufacture of grain mill products,Manufacture of other paper products,50.02298453
Manufacture of grain mill products,Printing,25.73962414
Manufacture of grain mill products,Manufacture of refined petroleum products,54.83857947
Manufacture of grain mill products,Manufacture of basic chemicals,705.7045932
Manufacture of grain mill products,Manufacture of paints,15.22534731
Manufacture of grain mill products,Manufacture of pharmaceutical products,687.2841957
Manufacture of grain mill products,Manufacture of soap detergents and toilet products,85.61726687
Manufacture of grain mill products,Manufacture of other chemical products,31.97947944
Manufacture of grain mill products,Manufacture of rubber products,4.21361911
Manufacture of grain mill products,Manufacture of plastic products,64.49150721
Manufacture of grain mill products,Manufacture of glass and glass products,16.86657734
Manufacture of grain mill products,Manufacture of cement,40.58522763
Manufacture of grain mill products,Manufacture of concrete and concrete products,66.28850933
Manufacture of grain mill products,Manufacture of basic iron and steel,42.21399194
Manufacture of grain mill products,Manufacture of other basic metals,16.75991038
Manufacture of grain mill products,Manufacture of fabricated metal products,112.090516
Manufacture of grain mill products,Manufacture of industrial and domestic machinery and equipment,56.28853664
Manufacture of grain mill products,Manufacture of electric and electronic machinery and equipment,19.87141132
Manufacture of grain mill products,Manufacture of transport equipment,25.7715721
Manufacture of grain mill products,Manufacture of furniture,42.41852691
Manufacture of grain mill products,Repair and installation of machinery and other manufacturing,21.03317009
Manufacture of grain mill products,Electric power generation,31.76644225
Manufacture of grain mill products,Transmission of electric power,0.480237968
Manufacture of grain mill products,Distribution of electric power,1053.467185
Manufacture of grain mill products,Gas and steam manufacture and supply,71.87132928
Manufacture of grain mill products,Water collection treatment and supply,4.964120312
Manufacture of grain mill products,Waste collection and recycling activities,5.441764905
Manufacture of grain mill products,Construction of residential buildings,111.801276
Manufacture of grain mill products,Construction of non-residential buildings,55.37096494
Manufacture of grain mill products,Civil engineering,93.65250186
Manufacture of grain mill products,Specialized construction activities,90.65794037
Manufacture of grain mill products,Wholesale and retail trade of motor vehicles,97.88470564
Manufacture of grain mill products,Wholesale trade,537.3802885
Manufacture of grain mill products,Retail trade,135035.1747
Manufacture of grain mill products,Accommodation,3322.672537
Manufacture of grain mill products,Food and beverage service activities,44059.88862
Manufacture of grain mill products,Transport via railways,4.701417374
Manufacture of grain mill products,Other passenger land transport,28.65346151
Manufacture of grain mill products,Freight transport by road,68.38959346
Manufacture of grain mill products,Transport via pipeline,0.07486135
Manufacture of grain mill products,Water transport,4.956938108
Manufacture of grain mill products,Air transport,39.04870546
Manufacture of grain mill products,Warehousing,19.59771489
Manufacture of grain mill products,Support activities for land transport,17.63490142
Manufacture of grain mill products,Other support activities for transport,20.7529491
Manufacture of grain mill products,Postal and courier activities,2.551981156
Manufacture of grain mill products,Wireless telecommunications,109.7539309
Manufacture of grain mill products,Wired telecommunications,45.42831843
Manufacture of grain mill products,Other telecommunications activities,79.76818266
Manufacture of grain mill products,Information services activities,178.4197861
Manufacture of grain mill products,Publishing activities,50.0085517
Manufacture of grain mill products,Banking monetary intermediation,230.3810913
Manufacture of grain mill products,Insurance activities,109.2739014
Manufacture of grain mill products,Auxiliary financial activities,56.77686854
Manufacture of grain mill products,Real estate activities,90.12846176
Manufacture of grain mill products,Housing services,0
Manufacture of grain mill products,Legal and accounting services,63.22093311
Manufacture of grain mill products,Architectural and engineering services,140.0915752
Manufacture of grain mill products,Other professional activities,310.1926988
Manufacture of grain mill products,Rental and leasing activities,54.13810086
Manufacture of grain mill products,Business support activities,250.7475351
Manufacture of grain mill products,Public administration,2633.963166
Manufacture of grain mill products,Public education,92.73696292
Manufacture of grain mill products,Private education,202.0544467
Manufacture of grain mill products,Public health activities,574.2464691
Manufacture of grain mill products,Private health activities,1858.722411
Manufacture of grain mill products,Activities of organizations,1785.667231
Manufacture of grain mill products,Artistic and entertainment activities,33.63591752
Manufacture of grain mill products,Other personal services,11.36779784
Manufacture of prepared animal feeds,Cultivation of annual crops,50.2576481
Manufacture of prepared animal feeds,Cultivation of vegetables,13.09238708
Manufacture of prepared animal feeds,Cultivation of grapes,28.06463418
Manufacture of prepared animal feeds,Cultivation of other fruits,1005.042528
Manufacture of prepared animal feeds,Cattle bredding,32647.0703
Manufacture of prepared animal feeds,Pigs breeding,216294.5763
Manufacture of prepared animal feeds,Chicken breeding,469648.7719
Manufacture of prepared animal feeds,Breeding of other animals,7612.804002
Manufacture of prepared animal feeds,Farming services,17.3449664
Manufacture of prepared animal feeds,Forestry,192.3932211
Manufacture of prepared animal feeds,Aquaculture,1006236.834
Manufacture of prepared animal feeds,Extractive fishing,8.229162618
Manufacture of prepared animal feeds,Coal mining,1.489959056
Manufacture of prepared animal feeds,Crude oil and gas mining,0.583129566
Manufacture of prepared animal feeds,Copper mining,258.0355283
Manufacture of prepared animal feeds,Iron mining,4.871790509
Manufacture of prepared animal feeds,Mining of other metals,8.907919664
Manufacture of prepared animal feeds,Other mining activities and mining services,9.394384146
Manufacture of prepared animal feeds,Processing and preserving of meat,79.36225642
Manufacture of prepared animal feeds,Processing and preserving of fishmeal and fish oil,4.948072881
Manufacture of prepared animal feeds,Processing and preserving of fish,43.3795419
Manufacture of prepared animal feeds,Processing and preserving of fruits and vegetables,33.37269249
Manufacture of prepared animal feeds,Manufacture of vegetable and animal oils,11.30181853
Manufacture of prepared animal feeds,Manufacture of dairy products,548.4968653
Manufacture of prepared animal feeds,Manufacture of grain mill products,24.99219923
Manufacture of prepared animal feeds,Manufacture of prepared animal feeds,58.26965623
Manufacture of prepared animal feeds,Manufacture of bakery products,47.32573018
Manufacture of prepared animal feeds,Manufacture of macaroni noodles and other related products,2.433159425
Manufacture of prepared animal feeds,Manufacture of other food products,53.0486747
Manufacture of prepared animal feeds,Distilling rectifying and blending of spirits,9.129030699
Manufacture of prepared animal feeds,Manufacture of wines,71.62325205
Manufacture of prepared animal feeds,Manufacture of beers,32.96024071
Manufacture of prepared animal feeds,Manufacture of soft drinks,78.6463818
Manufacture of prepared animal feeds,Manufacture of tobacco products,6.920747742
Manufacture of prepared animal feeds,Manufacture of textile products,19.89773679
Manufacture of prepared animal feeds,Manufacture of wearing apparel,54.378005
Manufacture of prepared animal feeds,Manufacture of leather products,1.357534228
Manufacture of prepared animal feeds,Manufacture of footwear,4.603903549
Manufacture of prepared animal feeds,Sawmilling and planing of wood,43.10190682
Manufacture of prepared animal feeds,Manufacture of wood products,42.8571638
Manufacture of prepared animal feeds,Manufacture of pulp and paper,60.56360459
Manufacture of prepared animal feeds,Manufacture of containers of paper,19.14727149
Manufacture of prepared animal feeds,Manufacture of other paper products,44.06263997
Manufacture of prepared animal feeds,Printing,24.89668591
Manufacture of prepared animal feeds,Manufacture of refined petroleum products,34.24157622
Manufacture of prepared animal feeds,Manufacture of basic chemicals,56.35829712
Manufacture of prepared animal feeds,Manufacture of paints,16.29024985
Manufacture of prepared animal feeds,Manufacture of pharmaceutical products,87.06686365
Manufacture of prepared animal feeds,Manufacture of soap detergents and toilet products,47.87677065
Manufacture of prepared animal feeds,Manufacture of other chemical products,25.51481143
Manufacture of prepared animal feeds,Manufacture of rubber products,5.274888328
Manufacture of prepared animal feeds,Manufacture of plastic products,60.94913431
Manufacture of prepared animal feeds,Manufacture of glass and glass products,15.11566145
Manufacture of prepared animal feeds,Manufacture of cement,25.68992161
Manufacture of prepared animal feeds,Manufacture of concrete and concrete products,68.87165906
Manufacture of prepared animal feeds,Manufacture of basic iron and steel,47.00036389
Manufacture of prepared animal feeds,Manufacture of other basic metals,22.79138882
Manufacture of prepared animal feeds,Manufacture of fabricated metal products,104.5300203
Manufacture of prepared animal feeds,Manufacture of industrial and domestic machinery and equipment,65.10436116
Manufacture of prepared animal feeds,Manufacture of electric and electronic machinery and equipment,25.399978
Manufacture of prepared animal feeds,Manufacture of transport equipment,24.77591917
Manufacture of prepared animal feeds,Manufacture of furniture,51.29330208
Manufacture of prepared animal feeds,Repair and installation of machinery and other manufacturing,25.5299179
Manufacture of prepared animal feeds,Electric power generation,35.0002788
Manufacture of prepared animal feeds,Transmission of electric power,0.478364808
Manufacture of prepared animal feeds,Distribution of electric power,6.428191596
Manufacture of prepared animal feeds,Gas and steam manufacture and supply,52.25830694
Manufacture of prepared animal feeds,Water collection treatment and supply,4.729054333
Manufacture of prepared animal feeds,Waste collection and recycling activities,6.356449776
Manufacture of prepared animal feeds,Construction of residential buildings,136.7875055
Manufacture of prepared animal feeds,Construction of non-residential buildings,77.45491616
Manufacture of prepared animal feeds,Civil engineering,158.9909939
Manufacture of prepared animal feeds,Specialized construction activities,165.6935484
Manufacture of prepared animal feeds,Wholesale and retail trade of motor vehicles,120.3181463
Manufacture of prepared animal feeds,Wholesale trade,563.5211256
Manufacture of prepared animal feeds,Retail trade,1464.626266
Manufacture of prepared animal feeds,Accommodation,19.18765239
Manufacture of prepared animal feeds,Food and beverage service activities,98.4283216
Manufacture of prepared animal feeds,Transport via railways,4.425636594
Manufacture of prepared animal feeds,Other passenger land transport,44.04563906
Manufacture of prepared animal feeds,Freight transport by road,98.87141378
Manufacture of prepared animal feeds,Transport via pipeline,0.084047785
Manufacture of prepared animal feeds,Water transport,6.47658068
Manufacture of prepared animal feeds,Air transport,41.3072245
Manufacture of prepared animal feeds,Warehousing,9.096473336
Manufacture of prepared animal feeds,Support activities for land transport,14.56641747
Manufacture of prepared animal feeds,Other support activities for transport,27.41112219
Manufacture of prepared animal feeds,Postal and courier activities,2.831571569
Manufacture of prepared animal feeds,Wireless telecommunications,152.2583622
Manufacture of prepared animal feeds,Wired telecommunications,57.84078017
Manufacture of prepared animal feeds,Other telecommunications activities,85.37577111
Manufacture of prepared animal feeds,Information services activities,178.340852
Manufacture of prepared animal feeds,Publishing activities,48.20382498
Manufacture of prepared animal feeds,Banking monetary intermediation,213.2794894
Manufacture of prepared animal feeds,Insurance activities,93.38369405
Manufacture of prepared animal feeds,Auxiliary financial activities,53.11129322
Manufacture of prepared animal feeds,Real estate activities,77.02138004
Manufacture of prepared animal feeds,Housing services,0
Manufacture of prepared animal feeds,Legal and accounting services,51.03941203
Manufacture of prepared animal feeds,Architectural and engineering services,122.376144
Manufacture of prepared animal feeds,Other professional activities,6079.764274
Manufacture of prepared animal feeds,Rental and leasing activities,54.78214188
Manufacture of prepared animal feeds,Business support activities,244.9724702
Manufacture of prepared animal feeds,Public administration,1403.483792
Manufacture of prepared animal feeds,Public education,115.8330323
Manufacture of prepared animal feeds,Private education,341.9891676
Manufacture of prepared animal feeds,Public health activities,247.0371019
Manufacture of prepared animal feeds,Private health activities,151.1684382
Manufacture of prepared animal feeds,Activities of organizations,15.48784197
Manufacture of prepared animal feeds,Artistic and entertainment activities,406.2065373
Manufacture of prepared animal feeds,Other personal services,18.59117131
Manufacture of bakery products,Cultivation of annual crops,684.3937305
Manufacture of bakery products,Cultivation of vegetables,175.8227189
Manufacture of bakery products,Cultivation of grapes,415.364328
Manufacture of bakery products,Cultivation of other fruits,1034.233453
Manufacture of bakery products,Cattle bredding,789.9404786
Manufacture of bakery products,Pigs breeding,99.53050254
Manufacture of bakery products,Chicken breeding,131.4676696
Manufacture of bakery products,Breeding of other animals,119.0945051
Manufacture of bakery products,Farming services,251.7263658
Manufacture of bakery products,Forestry,201.3134422
Manufacture of bakery products,Aquaculture,273.9745187
Manufacture of bakery products,Extractive fishing,582.6955671
Manufacture of bakery products,Coal mining,20.84440195
Manufacture of bakery products,Crude oil and gas mining,4.497763057
Manufacture of bakery products,Copper mining,2025.045388
Manufacture of bakery products,Iron mining,67.68642299
Manufacture of bakery products,Mining of other metals,119.9081144
Manufacture of bakery products,Other mining activities and mining services,63.11718974
Manufacture of bakery products,Processing and preserving of meat,662.9291069
Manufacture of bakery products,Processing and preserving of fishmeal and fish oil,36.3867111
Manufacture of bakery products,Processing and preserving of fish,167.8393178
Manufacture of bakery products,Processing and preserving of fruits and vegetables,389.1695768
Manufacture of bakery products,Manufacture of vegetable and animal oils,139.3606233
Manufacture of bakery products,Manufacture of dairy products,801.5125045
Manufacture of bakery products,Manufacture of grain mill products,267.6925726
Manufacture of bakery products,Manufacture of prepared animal feeds,2473.942201
Manufacture of bakery products,Manufacture of bakery products,6266.675765
Manufacture of bakery products,Manufacture of macaroni noodles and other related products,1134.505026
Manufacture of bakery products,Manufacture of other food products,11178.27183
Manufacture of bakery products,Distilling rectifying and blending of spirits,90.36095933
Manufacture of bakery products,Manufacture of wines,327.8296755
Manufacture of bakery products,Manufacture of beers,669.1958283
Manufacture of bakery products,Manufacture of soft drinks,1578.878423
Manufacture of bakery products,Manufacture of tobacco products,116.2770738
Manufacture of bakery products,Manufacture of textile products,151.6302775
Manufacture of bakery products,Manufacture of wearing apparel,403.4592184
Manufacture of bakery products,Manufacture of leather products,12.43466621
Manufacture of bakery products,Manufacture of footwear,40.75856182
Manufacture of bakery products,Sawmilling and planing of wood,73.17026401
Manufacture of bakery products,Manufacture of wood products,143.6339947
Manufacture of bakery products,Manufacture of pulp and paper,329.2951257
Manufacture of bakery products,Manufacture of containers of paper,116.1393303
Manufacture of bakery products,Manufacture of other paper products,273.6328878
Manufacture of bakery products,Printing,111.2255834
Manufacture of bakery products,Manufacture of refined petroleum products,89.88840877
Manufacture of bakery products,Manufacture of basic chemicals,244.2874901
Manufacture of bakery products,Manufacture of paints,103.7806281
Manufacture of bakery products,Manufacture of pharmaceutical products,291.2127758
Manufacture of bakery products,Manufacture of soap detergents and toilet products,363.1417621
Manufacture of bakery products,Manufacture of other chemical products,216.4038654
Manufacture of bakery products,Manufacture of rubber products,66.4525307
Manufacture of bakery products,Manufacture of plastic products,655.7890403
Manufacture of bakery products,Manufacture of glass and glass products,148.0864471
Manufacture of bakery products,Manufacture of cement,138.9383563
Manufacture of bakery products,Manufacture of concrete and concrete products,440.0177458
Manufacture of bakery products,Manufacture of basic iron and steel,553.7671285
Manufacture of bakery products,Manufacture of other basic metals,258.5561083
Manufacture of bakery products,Manufacture of fabricated metal products,691.8939739
Manufacture of bakery products,Manufacture of industrial and domestic machinery and equipment,625.583298
Manufacture of bakery products,Manufacture of electric and electronic machinery and equipment,233.5935913
Manufacture of bakery products,Manufacture of transport equipment,160.3073774
Manufacture of bakery products,Manufacture of furniture,436.983911
Manufacture of bakery products,Repair and installation of machinery and other manufacturing,228.7405224
Manufacture of bakery products,Electric power generation,270.8924508
Manufacture of bakery products,Transmission of electric power,0.922855584
Manufacture of bakery products,Distribution of electric power,6.009354533
Manufacture of bakery products,Gas and steam manufacture and supply,77.16492646
Manufacture of bakery products,Water collection treatment and supply,17.12164877
Manufacture of bakery products,Waste collection and recycling activities,32.6968931
Manufacture of bakery products,Construction of residential buildings,1274.302299
Manufacture of bakery products,Construction of non-residential buildings,894.422523
Manufacture of bakery products,Civil engineering,2280.802065
Manufacture of bakery products,Specialized construction activities,2371.489866
Manufacture of bakery products,Wholesale and retail trade of motor vehicles,1078.862363
Manufacture of bakery products,Wholesale trade,3309.935953
Manufacture of bakery products,Retail trade,10843.90271
Manufacture of bakery products,Accommodation,3992.262767
Manufacture of bakery products,Food and beverage service activities,95215.61741
Manufacture of bakery products,Transport via railways,18.40701548
Manufacture of bakery products,Other passenger land transport,543.3271124
Manufacture of bakery products,Freight transport by road,717.9225459
Manufacture of bakery products,Transport via pipeline,0.154240345
Manufacture of bakery products,Water transport,57.67931576
Manufacture of bakery products,Air transport,196.4754555
Manufacture of bakery products,Warehousing,91.37511959
Manufacture of bakery products,Support activities for land transport,22.81382002
Manufacture of bakery products,Other support activities for transport,216.1506403
Manufacture of bakery products,Postal and courier activities,6.538076681
Manufacture of bakery products,Wireless telecommunications,1662.958672
Manufacture of bakery products,Wired telecommunications,423.2181947
Manufacture of bakery products,Other telecommunications activities,610.2379634
Manufacture of bakery products,Information services activities,309.0571101
Manufacture of bakery products,Publishing activities,238.360497
Manufacture of bakery products,Banking monetary intermediation,371.9783678
Manufacture of bakery products,Insurance activities,164.8485052
Manufacture of bakery products,Auxiliary financial activities,137.1315764
Manufacture of bakery products,Real estate activities,231.6988435
Manufacture of bakery products,Housing services,0
Manufacture of bakery products,Legal and accounting services,77.4382441
Manufacture of bakery products,Architectural and engineering services,297.7136212
Manufacture of bakery products,Other professional activities,569.8068296
Manufacture of bakery products,Rental and leasing activities,286.7792704
Manufacture of bakery products,Business support activities,1156.4684
Manufacture of bakery products,Public administration,8550.500191
Manufacture of bakery products,Public education,304.1881247
Manufacture of bakery products,Private education,757.5634762
Manufacture of bakery products,Public health activities,2474.08779
Manufacture of bakery products,Private health activities,3133.93797
Manufacture of bakery products,Activities of organizations,2256.733631
Manufacture of bakery products,Artistic and entertainment activities,259.8478259
Manufacture of bakery products,Other personal services,233.2246716
Manufacture of macaroni noodles and other related products,Cultivation of annual crops,112.4457355
Manufacture of macaroni noodles and other related products,Cultivation of vegetables,28.15649546
Manufacture of macaroni noodles and other related products,Cultivation of grapes,61.94712297
Manufacture of macaroni noodles and other related products,Cultivation of other fruits,149.1926643
Manufacture of macaroni noodles and other related products,Cattle bredding,71.406479
Manufacture of macaroni noodles and other related products,Pigs breeding,11.19754275
Manufacture of macaroni noodles and other related products,Chicken breeding,12.75054515
Manufacture of macaroni noodles and other related products,Breeding of other animals,7.128253974
Manufacture of macaroni noodles and other related products,Farming services,41.30543972
Manufacture of macaroni noodles and other related products,Forestry,28.71038733
Manufacture of macaroni noodles and other related products,Aquaculture,45.4182218
Manufacture of macaroni noodles and other related products,Extractive fishing,606.2171997
Manufacture of macaroni noodles and other related products,Coal mining,3.472806692
Manufacture of macaroni noodles and other related products,Crude oil and gas mining,0.742546801
Manufacture of macaroni noodles and other related products,Copper mining,323.9277246
Manufacture of macaroni noodles and other related products,Iron mining,11.29648673
Manufacture of macaroni noodles and other related products,Mining of other metals,19.99252357
Manufacture of macaroni noodles and other related products,Other mining activities and mining services,10.37562116
Manufacture of macaroni noodles and other related products,Processing and preserving of meat,60.93708129
Manufacture of macaroni noodles and other related products,Processing and preserving of fishmeal and fish oil,5.488960708
Manufacture of macaroni noodles and other related products,Processing and preserving of fish,17.1734933
Manufacture of macaroni noodles and other related products,Processing and preserving of fruits and vegetables,33.54875234
Manufacture of macaroni noodles and other related products,Manufacture of vegetable and animal oils,15.63977099
Manufacture of macaroni noodles and other related products,Manufacture of dairy products,55.92284467
Manufacture of macaroni noodles and other related products,Manufacture of grain mill products,12.35591973
Manufacture of macaroni noodles and other related products,Manufacture of prepared animal feeds,83.93269609
Manufacture of macaroni noodles and other related products,Manufacture of bakery products,50.51386535
Manufacture of macaroni noodles and other related products,Manufacture of macaroni noodles and other related products,241.1266954
Manufacture of macaroni noodles and other related products,Manufacture of other food products,46.07887808
Manufacture of macaroni noodles and other related products,Distilling rectifying and blending of spirits,3.138890937
Manufacture of macaroni noodles and other related products,Manufacture of wines,32.23769809
Manufacture of macaroni noodles and other related products,Manufacture of beers,13.54216071
Manufacture of macaroni noodles and other related products,Manufacture of soft drinks,38.37258992
Manufacture of macaroni noodles and other related products,Manufacture of tobacco products,1.340094007
Manufacture of macaroni noodles and other related products,Manufacture of textile products,23.80254565
Manufacture of macaroni noodles and other related products,Manufacture of wearing apparel,63.12278905
Manufacture of macaroni noodles and other related products,Manufacture of leather products,1.838287766
Manufacture of macaroni noodles and other related products,Manufacture of footwear,6.368938206
Manufacture of macaroni noodles and other related products,Sawmilling and planing of wood,5.487293305
Manufacture of macaroni noodles and other related products,Manufacture of wood products,17.43115008
Manufacture of macaroni noodles and other related products,Manufacture of pulp and paper,48.93265379
Manufacture of macaroni noodles and other related products,Manufacture of containers of paper,17.00146361
Manufacture of macaroni noodles and other related products,Manufacture of other paper products,41.10573095
Manufacture of macaroni noodles and other related products,Printing,15.23701801
Manufacture of macaroni noodles and other related products,Manufacture of refined petroleum products,10.90502859
Manufacture of macaroni noodles and other related products,Manufacture of basic chemicals,32.04828528
Manufacture of macaroni noodles and other related products,Manufacture of paints,15.66114315
Manufacture of macaroni noodles and other related products,Manufacture of pharmaceutical products,27.1711179
Manufacture of macaroni noodles and other related products,Manufacture of soap detergents and toilet products,32.9906368
Manufacture of macaroni noodles and other related products,Manufacture of other chemical products,22.99315419
Manufacture of macaroni noodles and other related products,Manufacture of rubber products,11.06009362
Manufacture of macaroni noodles and other related products,Manufacture of plastic products,106.166664
Manufacture of macaroni noodles and other related products,Manufacture of glass and glass products,24.09745397
Manufacture of macaroni noodles and other related products,Manufacture of cement,20.24863862
Manufacture of macaroni noodles and other related products,Manufacture of concrete and concrete products,65.19561815
Manufacture of macaroni noodles and other related products,Manufacture of basic iron and steel,90.47193716
Manufacture of macaroni noodles and other related products,Manufacture of other basic metals,42.4362037
Manufacture of macaroni noodles and other related products,Manufacture of fabricated metal products,106.6944733
Manufacture of macaroni noodles and other related products,Manufacture of industrial and domestic machinery and equipment,99.39284352
Manufacture of macaroni noodles and other related products,Manufacture of electric and electronic machinery and equipment,37.66398093
Manufacture of macaroni noodles and other related products,Manufacture of transport equipment,23.86685122
Manufacture of macaroni noodles and other related products,Manufacture of furniture,71.28978484
Manufacture of macaroni noodles and other related products,Repair and installation of machinery and other manufacturing,36.12346134
Manufacture of macaroni noodles and other related products,Electric power generation,41.25774725
Manufacture of macaroni noodles and other related products,Transmission of electric power,0.137572492
Manufacture of macaroni noodles and other related products,Distribution of electric power,1.140691797
Manufacture of macaroni noodles and other related products,Gas and steam manufacture and supply,0.726854975
Manufacture of macaroni noodles and other related products,Water collection treatment and supply,2.311220408
Manufacture of macaroni noodles and other related products,Waste collection and recycling activities,5.226682642
Manufacture of macaroni noodles and other related products,Construction of residential buildings,200.9715053
Manufacture of macaroni noodles and other related products,Construction of non-residential buildings,143.3277217
Manufacture of macaroni noodles and other related products,Civil engineering,368.0048472
Manufacture of macaroni noodles and other related products,Specialized construction activities,395.1534347
Manufacture of macaroni noodles and other related products,Wholesale and retail trade of motor vehicles,170.9854728
Manufacture of macaroni noodles and other related products,Wholesale trade,495.2224881
Manufacture of macaroni noodles and other related products,Retail trade,922.8973268
Manufacture of macaroni noodles and other related products,Accommodation,35.15167279
Manufacture of macaroni noodles and other related products,Food and beverage service activities,2335.510939
Manufacture of macaroni noodles and other related products,Transport via railways,2.442589176
Manufacture of macaroni noodles and other related products,Other passenger land transport,89.38662342
Manufacture of macaroni noodles and other related products,Freight transport by road,118.2051498
Manufacture of macaroni noodles and other related products,Transport via pipeline,0.030787881
Manufacture of macaroni noodles and other related products,Water transport,9.33852701
Manufacture of macaroni noodles and other related products,Air transport,30.2285172
Manufacture of macaroni noodles and other related products,Warehousing,0.415372205
Manufacture of macaroni noodles and other related products,Support activities for land transport,1.057560981
Manufacture of macaroni noodles and other related products,Other support activities for transport,36.19313975
Manufacture of macaroni noodles and other related products,Postal and courier activities,0.86439984
Manufacture of macaroni noodles and other related products,Wireless telecommunications,270.3169028
Manufacture of macaroni noodles and other related products,Wired telecommunications,70.49183912
Manufacture of macaroni noodles and other related products,Other telecommunications activities,91.82890554
Manufacture of macaroni noodles and other related products,Information services activities,47.10042688
Manufacture of macaroni noodles and other related products,Publishing activities,32.79039539
Manufacture of macaroni noodles and other related products,Banking monetary intermediation,43.04265976
Manufacture of macaroni noodles and other related products,Insurance activities,12.41720273
Manufacture of macaroni noodles and other related products,Auxiliary financial activities,17.43638374
Manufacture of macaroni noodles and other related products,Real estate activities,23.61068398
Manufacture of macaroni noodles and other related products,Housing services,0
Manufacture of macaroni noodles and other related products,Legal and accounting services,2.207112425
Manufacture of macaroni noodles and other related products,Architectural and engineering services,29.83196249
Manufacture of macaroni noodles and other related products,Other professional activities,54.02501257
Manufacture of macaroni noodles and other related products,Rental and leasing activities,41.42698592
Manufacture of macaroni noodles and other related products,Business support activities,161.9477693
Manufacture of macaroni noodles and other related products,Public administration,1636.344004
Manufacture of macaroni noodles and other related products,Public education,74.28367952
Manufacture of macaroni noodles and other related products,Private education,91.98990972
Manufacture of macaroni noodles and other related products,Public health activities,732.8559105
Manufacture of macaroni noodles and other related products,Private health activities,261.5006103
Manufacture of macaroni noodles and other related products,Activities of organizations,32.55375651
Manufacture of macaroni noodles and other related products,Artistic and entertainment activities,33.76145043
Manufacture of macaroni noodles and other related products,Other personal services,38.78217055
Manufacture of other food products,Cultivation of annual crops,629.808494
Manufacture of other food products,Cultivation of vegetables,180.8251879
Manufacture of other food products,Cultivation of grapes,387.4819612
Manufacture of other food products,Cultivation of other fruits,784.6924599
Manufacture of other food products,Cattle bredding,1685.565292
Manufacture of other food products,Pigs breeding,295.0395924
Manufacture of other food products,Chicken breeding,342.7689307
Manufacture of other food products,Breeding of other animals,319.8016627
Manufacture of other food products,Farming services,191.7136483
Manufacture of other food products,Forestry,579.4970365
Manufacture of other food products,Aquaculture,255.8043683
Manufacture of other food products,Extractive fishing,919.0886722
Manufacture of other food products,Coal mining,70.20171427
Manufacture of other food products,Crude oil and gas mining,5.682338826
Manufacture of other food products,Copper mining,7129.768877
Manufacture of other food products,Iron mining,116.6366647
Manufacture of other food products,Mining of other metals,202.846168
Manufacture of other food products,Other mining activities and mining services,107.0183429
Manufacture of other food products,Processing and preserving of meat,3090.424627
Manufacture of other food products,Processing and preserving of fishmeal and fish oil,32.40583663
Manufacture of other food products,Processing and preserving of fish,369.6757985
Manufacture of other food products,Processing and preserving of fruits and vegetables,2557.074473
Manufacture of other food products,Manufacture of vegetable and animal oils,752.9942246
Manufacture of other food products,Manufacture of dairy products,4740.985976
Manufacture of other food products,Manufacture of grain mill products,327.6709725
Manufacture of other food products,Manufacture of prepared animal feeds,8061.544783
Manufacture of other food products,Manufacture of bakery products,38607.27754
Manufacture of other food products,Manufacture of macaroni noodles and other related products,4759.740571
Manufacture of other food products,Manufacture of other food products,54989.28796
Manufacture of other food products,Distilling rectifying and blending of spirits,928.157465
Manufacture of other food products,Manufacture of wines,739.045447
Manufacture of other food products,Manufacture of beers,3438.078727
Manufacture of other food products,Manufacture of soft drinks,16584.41082
Manufacture of other food products,Manufacture of tobacco products,1498.110646
Manufacture of other food products,Manufacture of textile products,316.3410583
Manufacture of other food products,Manufacture of wearing apparel,307.9250955
Manufacture of other food products,Manufacture of leather products,54.81410312
Manufacture of other food products,Manufacture of footwear,50.07431663
Manufacture of other food products,Sawmilling and planing of wood,168.8974134
Manufacture of other food products,Manufacture of wood products,543.7986321
Manufacture of other food products,Manufacture of pulp and paper,950.9308745
Manufacture of other food products,Manufacture of containers of paper,194.7551967
Manufacture of other food products,Manufacture of other paper products,369.0307145
Manufacture of other food products,Printing,92.38596546
Manufacture of other food products,Manufacture of refined petroleum products,263.7789241
Manufacture of other food products,Manufacture of basic chemicals,699.799291
Manufacture of other food products,Manufacture of paints,138.3677642
Manufacture of other food products,Manufacture of pharmaceutical products,639.0861226
Manufacture of other food products,Manufacture of soap detergents and toilet products,1292.297157
Manufacture of other food products,Manufacture of other chemical products,1384.954598
Manufacture of other food products,Manufacture of rubber products,97.61956068
Manufacture of other food products,Manufacture of plastic products,803.208955
Manufacture of other food products,Manufacture of glass and glass products,145.7344937
Manufacture of other food products,Manufacture of cement,151.919097
Manufacture of other food products,Manufacture of concrete and concrete products,759.3033641
Manufacture of other food products,Manufacture of basic iron and steel,448.9721173
Manufacture of other food products,Manufacture of other basic metals,199.7551437
Manufacture of other food products,Manufacture of fabricated metal products,598.4939403
Manufacture of other food products,Manufacture of industrial and domestic machinery and equipment,553.2687479
Manufacture of other food products,Manufacture of electric and electronic machinery and equipment,200.046893
Manufacture of other food products,Manufacture of transport equipment,122.0141306
Manufacture of other food products,Manufacture of furniture,482.8850343
Manufacture of other food products,Repair and installation of machinery and other manufacturing,182.6332999
Manufacture of other food products,Electric power generation,199.0907178
Manufacture of other food products,Transmission of electric power,1.016863741
Manufacture of other food products,Distribution of electric power,2715.984565
Manufacture of other food products,Gas and steam manufacture and supply,48.00440057
Manufacture of other food products,Water collection treatment and supply,40.37869867
Manufacture of other food products,Waste collection and recycling activities,31.72857575
Manufacture of other food products,Construction of residential buildings,975.8920553
Manufacture of other food products,Construction of non-residential buildings,749.1507389
Manufacture of other food products,Civil engineering,3732.64569
Manufacture of other food products,Specialized construction activities,2211.108203
Manufacture of other food products,Wholesale and retail trade of motor vehicles,796.6400746
Manufacture of other food products,Wholesale trade,2504.343432
Manufacture of other food products,Retail trade,42297.36904
Manufacture of other food products,Accommodation,4555.799833
Manufacture of other food products,Food and beverage service activities,62581.94582
Manufacture of other food products,Transport via railways,13.21320533
Manufacture of other food products,Other passenger land transport,403.9905756
Manufacture of other food products,Freight transport by road,567.9588803
Manufacture of other food products,Transport via pipeline,0.220648214
Manufacture of other food products,Water transport,43.75592134
Manufacture of other food products,Air transport,159.8053145
Manufacture of other food products,Warehousing,18.19288198
Manufacture of other food products,Support activities for land transport,14.02948693
Manufacture of other food products,Other support activities for transport,174.678316
Manufacture of other food products,Postal and courier activities,5.651884902
Manufacture of other food products,Wireless telecommunications,1235.048779
Manufacture of other food products,Wired telecommunications,344.1011158
Manufacture of other food products,Other telecommunications activities,439.7736998
Manufacture of other food products,Information services activities,365.5003041
Manufacture of other food products,Publishing activities,168.5735879
Manufacture of other food products,Banking monetary intermediation,357.2651999
Manufacture of other food products,Insurance activities,117.5520243
Manufacture of other food products,Auxiliary financial activities,134.7221537
Manufacture of other food products,Real estate activities,145.9435819
Manufacture of other food products,Housing services,0
Manufacture of other food products,Legal and accounting services,41.03528885
Manufacture of other food products,Architectural and engineering services,318.551305
Manufacture of other food products,Other professional activities,422.904316
Manufacture of other food products,Rental and leasing activities,211.1183313
Manufacture of other food products,Business support activities,848.9315018
Manufacture of other food products,Public administration,7051.07811
Manufacture of other food products,Public education,407.144377
Manufacture of other food products,Private education,351.4869986
Manufacture of other food products,Public health activities,2890.213692
Manufacture of other food products,Private health activities,2533.047294
Manufacture of other food products,Activities of organizations,1682.503467
Manufacture of other food products,Artistic and entertainment activities,169.5191414
Manufacture of other food products,Other personal services,175.4733159
Distilling rectifying and blending of spirits,Cultivation of annual crops,86.08274372
Distilling rectifying and blending of spirits,Cultivation of vegetables,21.61968966
Distilling rectifying and blending of spirits,Cultivation of grapes,47.45626183
Distilling rectifying and blending of spirits,Cultivation of other fruits,114.312163
Distilling rectifying and blending of spirits,Cattle bredding,54.89816551
Distilling rectifying and blending of spirits,Pigs breeding,8.723449992
Distilling rectifying and blending of spirits,Chicken breeding,10.06715561
Distilling rectifying and blending of spirits,Breeding of other animals,5.477911232
Distilling rectifying and blending of spirits,Farming services,31.50828374
Distilling rectifying and blending of spirits,Forestry,22.00315675
Distilling rectifying and blending of spirits,Aquaculture,34.81389502
Distilling rectifying and blending of spirits,Extractive fishing,15.08466531
Distilling rectifying and blending of spirits,Coal mining,2.651389143
Distilling rectifying and blending of spirits,Crude oil and gas mining,0.629618037
Distilling rectifying and blending of spirits,Copper mining,270.8332271
Distilling rectifying and blending of spirits,Iron mining,8.617675941
Distilling rectifying and blending of spirits,Mining of other metals,15.28692532
Distilling rectifying and blending of spirits,Other mining activities and mining services,77.37536109
Distilling rectifying and blending of spirits,Processing and preserving of meat,177.4533178
Distilling rectifying and blending of spirits,Processing and preserving of fishmeal and fish oil,4.554853025
Distilling rectifying and blending of spirits,Processing and preserving of fish,115.1286395
Distilling rectifying and blending of spirits,Processing and preserving of fruits and vegetables,28.26041788
Distilling rectifying and blending of spirits,Manufacture of vegetable and animal oils,12.57230315
Distilling rectifying and blending of spirits,Manufacture of dairy products,49.41202929
Distilling rectifying and blending of spirits,Manufacture of grain mill products,33.20027915
Distilling rectifying and blending of spirits,Manufacture of prepared animal feeds,67.57434028
Distilling rectifying and blending of spirits,Manufacture of bakery products,43.32496319
Distilling rectifying and blending of spirits,Manufacture of macaroni noodles and other related products,3.547227045
Distilling rectifying and blending of spirits,Manufacture of other food products,129.437024
Distilling rectifying and blending of spirits,Distilling rectifying and blending of spirits,9379.172717
Distilling rectifying and blending of spirits,Manufacture of wines,142.0945788
Distilling rectifying and blending of spirits,Manufacture of beers,14.3166307
Distilling rectifying and blending of spirits,Manufacture of soft drinks,39.48074426
Distilling rectifying and blending of spirits,Manufacture of tobacco products,1.960872703
Distilling rectifying and blending of spirits,Manufacture of textile products,20.03162461
Distilling rectifying and blending of spirits,Manufacture of wearing apparel,53.53018804
Distilling rectifying and blending of spirits,Manufacture of leather products,1.486892803
Distilling rectifying and blending of spirits,Manufacture of footwear,5.139615127
Distilling rectifying and blending of spirits,Sawmilling and planing of wood,10.69985344
Distilling rectifying and blending of spirits,Manufacture of wood products,18.38410499
Distilling rectifying and blending of spirits,Manufacture of pulp and paper,42.85849997
Distilling rectifying and blending of spirits,Manufacture of containers of paper,14.75411382
Distilling rectifying and blending of spirits,Manufacture of other paper products,36.13767666
Distilling rectifying and blending of spirits,Printing,29.55226619
Distilling rectifying and blending of spirits,Manufacture of refined petroleum products,13.33549234
Distilling rectifying and blending of spirits,Manufacture of basic chemicals,29.44729487
Distilling rectifying and blending of spirits,Manufacture of paints,507.5889528
Distilling rectifying and blending of spirits,Manufacture of pharmaceutical products,120.633023
Distilling rectifying and blending of spirits,Manufacture of soap detergents and toilet products,31.43909644
Distilling rectifying and blending of spirits,Manufacture of other chemical products,19.85100311
Distilling rectifying and blending of spirits,Manufacture of rubber products,8.544621969
Distilling rectifying and blending of spirits,Manufacture of plastic products,83.25489687
Distilling rectifying and blending of spirits,Manufacture of glass and glass products,19.34210889
Distilling rectifying and blending of spirits,Manufacture of cement,17.81300955
Distilling rectifying and blending of spirits,Manufacture of concrete and concrete products,69.24230346
Distilling rectifying and blending of spirits,Manufacture of basic iron and steel,70.27403482
Distilling rectifying and blending of spirits,Manufacture of other basic metals,33.33536406
Distilling rectifying and blending of spirits,Manufacture of fabricated metal products,92.64842034
Distilling rectifying and blending of spirits,Manufacture of industrial and domestic machinery and equipment,124.4419867
Distilling rectifying and blending of spirits,Manufacture of electric and electronic machinery and equipment,30.55190949
Distilling rectifying and blending of spirits,Manufacture of transport equipment,28.83177006
Distilling rectifying and blending of spirits,Manufacture of furniture,59.0453317
Distilling rectifying and blending of spirits,Repair and installation of machinery and other manufacturing,34.36355341
Distilling rectifying and blending of spirits,Electric power generation,34.1737501
Distilling rectifying and blending of spirits,Transmission of electric power,0.210446883
Distilling rectifying and blending of spirits,Distribution of electric power,2.565532887
Distilling rectifying and blending of spirits,Gas and steam manufacture and supply,8.409095862
Distilling rectifying and blending of spirits,Water collection treatment and supply,2.501929598
Distilling rectifying and blending of spirits,Waste collection and recycling activities,4.726445984
Distilling rectifying and blending of spirits,Construction of residential buildings,160.8493692
Distilling rectifying and blending of spirits,Construction of non-residential buildings,111.8737561
Distilling rectifying and blending of spirits,Civil engineering,281.5111798
Distilling rectifying and blending of spirits,Specialized construction activities,301.2776429
Distilling rectifying and blending of spirits,Wholesale and retail trade of motor vehicles,138.0022626
Distilling rectifying and blending of spirits,Wholesale trade,430.169558
Distilling rectifying and blending of spirits,Retail trade,740.2965242
Distilling rectifying and blending of spirits,Accommodation,6365.200747
Distilling rectifying and blending of spirits,Food and beverage service activities,4848.811138
Distilling rectifying and blending of spirits,Transport via railways,2.448336372
Distilling rectifying and blending of spirits,Other passenger land transport,69.25557431
Distilling rectifying and blending of spirits,Freight transport by road,91.80016688
Distilling rectifying and blending of spirits,Transport via pipeline,0.04551595
Distilling rectifying and blending of spirits,Water transport,7.54906412
Distilling rectifying and blending of spirits,Air transport,326.8213185
Distilling rectifying and blending of spirits,Warehousing,1.460939774
Distilling rectifying and blending of spirits,Support activities for land transport,3.230448995
Distilling rectifying and blending of spirits,Other support activities for transport,30.98393736
Distilling rectifying and blending of spirits,Postal and courier activities,0.800919085
Distilling rectifying and blending of spirits,Wireless telecommunications,212.9530782
Distilling rectifying and blending of spirits,Wired telecommunications,61.39684037
Distilling rectifying and blending of spirits,Other telecommunications activities,77.576417
Distilling rectifying and blending of spirits,Information services activities,76.84303814
Distilling rectifying and blending of spirits,Publishing activities,47.47283117
Distilling rectifying and blending of spirits,Banking monetary intermediation,76.1539809
Distilling rectifying and blending of spirits,Insurance activities,25.68230302
Distilling rectifying and blending of spirits,Auxiliary financial activities,22.95813629
Distilling rectifying and blending of spirits,Real estate activities,28.43612469
Distilling rectifying and blending of spirits,Housing services,0
Distilling rectifying and blending of spirits,Legal and accounting services,9.781435758
Distilling rectifying and blending of spirits,Architectural and engineering services,41.95400204
Distilling rectifying and blending of spirits,Other professional activities,87.40525135
Distilling rectifying and blending of spirits,Rental and leasing activities,37.89120704
Distilling rectifying and blending of spirits,Business support activities,154.7579327
Distilling rectifying and blending of spirits,Public administration,61.36645915
Distilling rectifying and blending of spirits,Public education,15.45417411
Distilling rectifying and blending of spirits,Private education,26.88377681
Distilling rectifying and blending of spirits,Public health activities,217.3163883
Distilling rectifying and blending of spirits,Private health activities,205.7611702
Distilling rectifying and blending of spirits,Activities of organizations,25.12599841
Distilling rectifying and blending of spirits,Artistic and entertainment activities,28.67406894
Distilling rectifying and blending of spirits,Other personal services,30.03337243
Manufacture of wines,Cultivation of annual crops,84.82811727
Manufacture of wines,Cultivation of vegetables,21.92326973
Manufacture of wines,Cultivation of grapes,47.81570939
Manufacture of wines,Cultivation of other fruits,113.3966576
Manufacture of wines,Cattle bredding,56.75477424
Manufacture of wines,Pigs breeding,10.37969017
Manufacture of wines,Chicken breeding,13.54460668
Manufacture of wines,Breeding of other animals,5.643806934
Manufacture of wines,Farming services,29.64795487
Manufacture of wines,Forestry,27.169451
Manufacture of wines,Aquaculture,37.44888253
Manufacture of wines,Extractive fishing,14.05550201
Manufacture of wines,Coal mining,2.550538742
Manufacture of wines,Crude oil and gas mining,1.478217778
Manufacture of wines,Copper mining,469.9343522
Manufacture of wines,Iron mining,8.602866799
Manufacture of wines,Mining of other metals,15.05142328
Manufacture of wines,Other mining activities and mining services,10.70862855
Manufacture of wines,Processing and preserving of meat,147.5476949
Manufacture of wines,Processing and preserving of fishmeal and fish oil,8.520036701
Manufacture of wines,Processing and preserving of fish,82.354705
Manufacture of wines,Processing and preserving of fruits and vegetables,85.54304125
Manufacture of wines,Manufacture of vegetable and animal oils,19.49108096
Manufacture of wines,Manufacture of dairy products,107.5340329
Manufacture of wines,Manufacture of grain mill products,45.92634346
Manufacture of wines,Manufacture of prepared animal feeds,102.5053505
Manufacture of wines,Manufacture of bakery products,85.19038418
Manufacture of wines,Manufacture of macaroni noodles and other related products,11.39162776
Manufacture of wines,Manufacture of other food products,1925.583366
Manufacture of wines,Distilling rectifying and blending of spirits,199.2603214
Manufacture of wines,Manufacture of wines,927.2234723
Manufacture of wines,Manufacture of beers,69.25829942
Manufacture of wines,Manufacture of soft drinks,134.9133679
Manufacture of wines,Manufacture of tobacco products,13.21467658
Manufacture of wines,Manufacture of textile products,36.28762577
Manufacture of wines,Manufacture of wearing apparel,99.60706468
Manufacture of wines,Manufacture of leather products,2.414073102
Manufacture of wines,Manufacture of footwear,8.131921384
Manufacture of wines,Sawmilling and planing of wood,75.57766349
Manufacture of wines,Manufacture of wood products,72.84568049
Manufacture of wines,Manufacture of pulp and paper,97.02456417
Manufacture of wines,Manufacture of containers of paper,33.7382704
Manufacture of wines,Manufacture of other paper products,79.67361059
Manufacture of wines,Printing,46.55258797
Manufacture of wines,Manufacture of refined petroleum products,59.24817789
Manufacture of wines,Manufacture of basic chemicals,76.74407319
Manufacture of wines,Manufacture of paints,38.75907754
Manufacture of wines,Manufacture of pharmaceutical products,167.1888221
Manufacture of wines,Manufacture of soap detergents and toilet products,88.28598758
Manufacture of wines,Manufacture of other chemical products,45.47168345
Manufacture of wines,Manufacture of rubber products,8.863228272
Manufacture of wines,Manufacture of plastic products,104.5552205
Manufacture of wines,Manufacture of glass and glass products,26.60210461
Manufacture of wines,Manufacture of cement,42.40873357
Manufacture of wines,Manufacture of concrete and concrete products,126.3173053
Manufacture of wines,Manufacture of basic iron and steel,81.31333426
Manufacture of wines,Manufacture of other basic metals,40.16352409
Manufacture of wines,Manufacture of fabricated metal products,191.2072217
Manufacture of wines,Manufacture of industrial and domestic machinery and equipment,113.2359464
Manufacture of wines,Manufacture of electric and electronic machinery and equipment,45.26326284
Manufacture of wines,Manufacture of transport equipment,45.76233218
Manufacture of wines,Manufacture of furniture,91.72844326
Manufacture of wines,Repair and installation of machinery and other manufacturing,45.88166089
Manufacture of wines,Electric power generation,63.62409507
Manufacture of wines,Transmission of electric power,0.981289444
Manufacture of wines,Distribution of electric power,13.78044016
Manufacture of wines,Gas and steam manufacture and supply,101.0135407
Manufacture of wines,Water collection treatment and supply,9.462137888
Manufacture of wines,Waste collection and recycling activities,10.42878669
Manufacture of wines,Construction of residential buildings,246.6808241
Manufacture of wines,Construction of non-residential buildings,138.9244536
Manufacture of wines,Civil engineering,288.924742
Manufacture of wines,Specialized construction activities,283.0772577
Manufacture of wines,Wholesale and retail trade of motor vehicles,211.3007683
Manufacture of wines,Wholesale trade,947.4513336
Manufacture of wines,Retail trade,1057.212743
Manufacture of wines,Accommodation,5651.252283
Manufacture of wines,Food and beverage service activities,27431.46729
Manufacture of wines,Transport via railways,8.296815112
Manufacture of wines,Other passenger land transport,77.82976393
Manufacture of wines,Freight transport by road,116.0723378
Manufacture of wines,Transport via pipeline,0.171333973
Manufacture of wines,Water transport,12.06587617
Manufacture of wines,Air transport,1870.854887
Manufacture of wines,Warehousing,14.9974298
Manufacture of wines,Support activities for land transport,28.76570394
Manufacture of wines,Other support activities for transport,54.30103944
Manufacture of wines,Postal and courier activities,2.549467154
Manufacture of wines,Wireless telecommunications,269.8121199
Manufacture of wines,Wired telecommunications,108.836179
Manufacture of wines,Other telecommunications activities,158.1610039
Manufacture of wines,Information services activities,356.1691506
Manufacture of wines,Publishing activities,89.96267128
Manufacture of wines,Banking monetary intermediation,419.2507342
Manufacture of wines,Insurance activities,181.5851166
Manufacture of wines,Auxiliary financial activities,103.2049959
Manufacture of wines,Real estate activities,146.8636778
Manufacture of wines,Housing services,0
Manufacture of wines,Legal and accounting services,100.921166
Manufacture of wines,Architectural and engineering services,235.9907539
Manufacture of wines,Other professional activities,526.1063836
Manufacture of wines,Rental and leasing activities,103.5034615
Manufacture of wines,Business support activities,458.3555693
Manufacture of wines,Public administration,716.028855
Manufacture of wines,Public education,73.28228352
Manufacture of wines,Private education,144.8145955
Manufacture of wines,Public health activities,266.4892405
Manufacture of wines,Private health activities,307.6858944
Manufacture of wines,Activities of organizations,103.7494085
Manufacture of wines,Artistic and entertainment activities,68.15991765
Manufacture of wines,Other personal services,32.01929728
Manufacture of beers,Cultivation of annual crops,127.7558064
Manufacture of beers,Cultivation of vegetables,32.10285121
Manufacture of beers,Cultivation of grapes,70.43710389
Manufacture of beers,Cultivation of other fruits,169.6708712
Manufacture of beers,Cattle bredding,81.52051695
Manufacture of beers,Pigs breeding,12.98207127
Manufacture of beers,Chicken breeding,1755.563368
Manufacture of beers,Breeding of other animals,8.133986262
Manufacture of beers,Farming services,46.72728168
Manufacture of beers,Forestry,36.04250367
Manufacture of beers,Aquaculture,53.21452476
Manufacture of beers,Extractive fishing,22.3711426
Manufacture of beers,Coal mining,3.942047899
Manufacture of beers,Crude oil and gas mining,1.214602544
Manufacture of beers,Copper mining,412.689923
Manufacture of beers,Iron mining,13.07135905
Manufacture of beers,Mining of other metals,22.8808761
Manufacture of beers,Other mining activities and mining services,12.26240431
Manufacture of beers,Processing and preserving of meat,83.03749877
Manufacture of beers,Processing and preserving of fishmeal and fish oil,6.843804185
Manufacture of beers,Processing and preserving of fish,28.74744655
Manufacture of beers,Processing and preserving of fruits and vegetables,42.70298794
Manufacture of beers,Manufacture of vegetable and animal oils,18.82555724
Manufacture of beers,Manufacture of dairy products,74.4283451
Manufacture of beers,Manufacture of grain mill products,51.61761797
Manufacture of beers,Manufacture of prepared animal feeds,101.2236384
Manufacture of beers,Manufacture of bakery products,65.14635857
Manufacture of beers,Manufacture of macaroni noodles and other related products,5.286440732
Manufacture of beers,Manufacture of other food products,62.31646608
Manufacture of beers,Distilling rectifying and blending of spirits,5.480551047
Manufacture of beers,Manufacture of wines,53.53150488
Manufacture of beers,Manufacture of beers,26741.1512
Manufacture of beers,Manufacture of soft drinks,60.48019633
Manufacture of beers,Manufacture of tobacco products,3.239256821
Manufacture of beers,Manufacture of textile products,30.07058447
Manufacture of beers,Manufacture of wearing apparel,80.31724979
Manufacture of beers,Manufacture of leather products,2.227409959
Manufacture of beers,Manufacture of footwear,7.693791766
Manufacture of beers,Sawmilling and planing of wood,17.23995594
Manufacture of beers,Manufacture of wood products,28.45498201
Manufacture of beers,Manufacture of pulp and paper,64.70983094
Manufacture of beers,Manufacture of containers of paper,22.33816369
Manufacture of beers,Manufacture of other paper products,54.47815698
Manufacture of beers,Printing,22.78116616
Manufacture of beers,Manufacture of refined petroleum products,20.67457062
Manufacture of beers,Manufacture of basic chemicals,1510.437925
Manufacture of beers,Manufacture of paints,20.53374877
Manufacture of beers,Manufacture of pharmaceutical products,52.71593362
Manufacture of beers,Manufacture of soap detergents and toilet products,47.75368378
Manufacture of beers,Manufacture of other chemical products,30.00489718
Manufacture of beers,Manufacture of rubber products,12.68432782
Manufacture of beers,Manufacture of plastic products,124.0626317
Manufacture of beers,Manufacture of glass and glass products,28.83776734
Manufacture of beers,Manufacture of cement,27.04310181
Manufacture of beers,Manufacture of concrete and concrete products,85.92150422
Manufacture of beers,Manufacture of basic iron and steel,104.6552766
Manufacture of beers,Manufacture of other basic metals,49.60302024
Manufacture of beers,Manufacture of fabricated metal products,139.6809945
Manufacture of beers,Manufacture of industrial and domestic machinery and equipment,118.1315707
Manufacture of beers,Manufacture of electric and electronic machinery and equipment,45.66328787
Manufacture of beers,Manufacture of transport equipment,31.23912493
Manufacture of beers,Manufacture of furniture,88.2037513
Manufacture of beers,Repair and installation of machinery and other manufacturing,44.08055014
Manufacture of beers,Electric power generation,51.53586259
Manufacture of beers,Transmission of electric power,0.347745143
Manufacture of beers,Distribution of electric power,4.429498086
Manufacture of beers,Gas and steam manufacture and supply,14.75157686
Manufacture of beers,Water collection treatment and supply,4.097605828
Manufacture of beers,Waste collection and recycling activities,7.328484768
Manufacture of beers,Construction of residential buildings,244.1648958
Manufacture of beers,Construction of non-residential buildings,168.5685795
Manufacture of beers,Civil engineering,429.8173914
Manufacture of beers,Specialized construction activities,447.382859
Manufacture of beers,Wholesale and retail trade of motor vehicles,206.8543538
Manufacture of beers,Wholesale trade,652.368542
Manufacture of beers,Retail trade,1106.968885
Manufacture of beers,Accommodation,1529.843312
Manufacture of beers,Food and beverage service activities,11862.838
Manufacture of beers,Transport via railways,3.768571895
Manufacture of beers,Other passenger land transport,103.6446132
Manufacture of beers,Freight transport by road,144.6992936
Manufacture of beers,Transport via pipeline,0.070279637
Manufacture of beers,Water transport,11.99958369
Manufacture of beers,Air transport,46.06732783
Manufacture of beers,Warehousing,2.576826548
Manufacture of beers,Support activities for land transport,5.63982374
Manufacture of beers,Other support activities for transport,48.4012736
Manufacture of beers,Postal and courier activities,1.285267309
Manufacture of beers,Wireless telecommunications,317.4205839
Manufacture of beers,Wired telecommunications,92.24786575
Manufacture of beers,Other telecommunications activities,117.8942623
Manufacture of beers,Information services activities,119.3302646
Manufacture of beers,Publishing activities,47.23463027
Manufacture of beers,Banking monetary intermediation,119.0689873
Manufacture of beers,Insurance activities,41.32276038
Manufacture of beers,Auxiliary financial activities,35.53342077
Manufacture of beers,Real estate activities,45.17706813
Manufacture of beers,Housing services,0
Manufacture of beers,Legal and accounting services,17.84286039
Manufacture of beers,Architectural and engineering services,68.29919202
Manufacture of beers,Other professional activities,139.8809821
Manufacture of beers,Rental and leasing activities,60.42851497
Manufacture of beers,Business support activities,237.6345115
Manufacture of beers,Public administration,396.8964286
Manufacture of beers,Public education,24.44741178
Manufacture of beers,Private education,42.47574186
Manufacture of beers,Public health activities,323.9963721
Manufacture of beers,Private health activities,307.6335347
Manufacture of beers,Activities of organizations,37.3386595
Manufacture of beers,Artistic and entertainment activities,44.90741613
Manufacture of beers,Other personal services,44.77070625
Manufacture of soft drinks,Cultivation of annual crops,365.9797824
Manufacture of soft drinks,Cultivation of vegetables,140.9834104
Manufacture of soft drinks,Cultivation of grapes,202.830772
Manufacture of soft drinks,Cultivation of other fruits,485.7281389
Manufacture of soft drinks,Cattle bredding,232.5685095
Manufacture of soft drinks,Pigs breeding,36.53786762
Manufacture of soft drinks,Chicken breeding,41.68595372
Manufacture of soft drinks,Breeding of other animals,23.21321091
Manufacture of soft drinks,Farming services,137.3559123
Manufacture of soft drinks,Forestry,93.52721204
Manufacture of soft drinks,Aquaculture,147.8728875
Manufacture of soft drinks,Extractive fishing,437.2756003
Manufacture of soft drinks,Coal mining,11.29671941
Manufacture of soft drinks,Crude oil and gas mining,2.54078349
Manufacture of soft drinks,Copper mining,1084.830748
Manufacture of soft drinks,Iron mining,36.74885308
Manufacture of soft drinks,Mining of other metals,65.08490134
Manufacture of soft drinks,Other mining activities and mining services,33.84188636
Manufacture of soft drinks,Processing and preserving of meat,203.7433307
Manufacture of soft drinks,Processing and preserving of fishmeal and fish oil,18.11076362
Manufacture of soft drinks,Processing and preserving of fish,59.20215869
Manufacture of soft drinks,Processing and preserving of fruits and vegetables,193.9419702
Manufacture of soft drinks,Manufacture of vegetable and animal oils,82.60502217
Manufacture of soft drinks,Manufacture of dairy products,487.2654521
Manufacture of soft drinks,Manufacture of grain mill products,164.2785647
Manufacture of soft drinks,Manufacture of prepared animal feeds,275.9845254
Manufacture of soft drinks,Manufacture of bakery products,390.6723147
Manufacture of soft drinks,Manufacture of macaroni noodles and other related products,14.87162635
Manufacture of soft drinks,Manufacture of other food products,421.9452203
Manufacture of soft drinks,Distilling rectifying and blending of spirits,39.12964337
Manufacture of soft drinks,Manufacture of wines,514.6151011
Manufacture of soft drinks,Manufacture of beers,49.11748708
Manufacture of soft drinks,Manufacture of soft drinks,205032.7316
Manufacture of soft drinks,Manufacture of tobacco products,4.920134781
Manufacture of soft drinks,Manufacture of textile products,79.60343599
Manufacture of soft drinks,Manufacture of wearing apparel,211.7094151
Manufacture of soft drinks,Manufacture of leather products,6.030594483
Manufacture of soft drinks,Manufacture of footwear,20.90280266
Manufacture of soft drinks,Sawmilling and planing of wood,24.3409005
Manufacture of soft drinks,Manufacture of wood products,60.86034063
Manufacture of soft drinks,Manufacture of pulp and paper,164.4669554
Manufacture of soft drinks,Manufacture of containers of paper,56.70258537
Manufacture of soft drinks,Manufacture of other paper products,138.9795339
Manufacture of soft drinks,Printing,52.93391106
Manufacture of soft drinks,Manufacture of refined petroleum products,41.24000395
Manufacture of soft drinks,Manufacture of basic chemicals,110.9442633
Manufacture of soft drinks,Manufacture of paints,52.66973439
Manufacture of soft drinks,Manufacture of pharmaceutical products,101.4973922
Manufacture of soft drinks,Manufacture of soap detergents and toilet products,212.5671746
Manufacture of soft drinks,Manufacture of other chemical products,77.44672571
Manufacture of soft drinks,Manufacture of rubber products,36.14315318
Manufacture of soft drinks,Manufacture of plastic products,359.5469736
Manufacture of soft drinks,Manufacture of glass and glass products,79.61065952
Manufacture of soft drinks,Manufacture of cement,67.87589984
Manufacture of soft drinks,Manufacture of concrete and concrete products,218.048014
Manufacture of soft drinks,Manufacture of basic iron and steel,295.1040253
Manufacture of soft drinks,Manufacture of other basic metals,139.2042668
Manufacture of soft drinks,Manufacture of fabricated metal products,360.3136302
Manufacture of soft drinks,Manufacture of industrial and domestic machinery and equipment,325.4831557
Manufacture of soft drinks,Manufacture of electric and electronic machinery and equipment,124.7314712
Manufacture of soft drinks,Manufacture of transport equipment,79.89347714
Manufacture of soft drinks,Manufacture of furniture,238.8032985
Manufacture of soft drinks,Repair and installation of machinery and other manufacturing,118.8530401
Manufacture of soft drinks,Electric power generation,141.4996096
Manufacture of soft drinks,Transmission of electric power,0.618230997
Manufacture of soft drinks,Distribution of electric power,6.739000152
Manufacture of soft drinks,Gas and steam manufacture and supply,7.325354333
Manufacture of soft drinks,Water collection treatment and supply,8.408514896
Manufacture of soft drinks,Waste collection and recycling activities,18.1174228
Manufacture of soft drinks,Construction of residential buildings,658.5289641
Manufacture of soft drinks,Construction of non-residential buildings,467.8105185
Manufacture of soft drinks,Civil engineering,1198.857664
Manufacture of soft drinks,Specialized construction activities,1285.394521
Manufacture of soft drinks,Wholesale and retail trade of motor vehicles,564.2566055
Manufacture of soft drinks,Wholesale trade,1698.571348
Manufacture of soft drinks,Retail trade,3225.472927
Manufacture of soft drinks,Accommodation,6641.959198
Manufacture of soft drinks,Food and beverage service activities,96765.8629
Manufacture of soft drinks,Transport via railways,11.78693423
Manufacture of soft drinks,Other passenger land transport,293.2307417
Manufacture of soft drinks,Freight transport by road,394.0731986
Manufacture of soft drinks,Transport via pipeline,0.14180328
Manufacture of soft drinks,Water transport,39.37113033
Manufacture of soft drinks,Air transport,223.2104809
Manufacture of soft drinks,Warehousing,73.77323322
Manufacture of soft drinks,Support activities for land transport,5.691142481
Manufacture of soft drinks,Other support activities for transport,136.4733792
Manufacture of soft drinks,Postal and courier activities,2.895853164
Manufacture of soft drinks,Wireless telecommunications,885.888027
Manufacture of soft drinks,Wired telecommunications,242.6458006
Manufacture of soft drinks,Other telecommunications activities,304.8504476
Manufacture of soft drinks,Information services activities,220.7489936
Manufacture of soft drinks,Publishing activities,111.211182
Manufacture of soft drinks,Banking monetary intermediation,201.3858843
Manufacture of soft drinks,Insurance activities,57.70689973
Manufacture of soft drinks,Auxiliary financial activities,69.54791956
Manufacture of soft drinks,Real estate activities,84.14889803
Manufacture of soft drinks,Housing services,0
Manufacture of soft drinks,Legal and accounting services,13.5798053
Manufacture of soft drinks,Architectural and engineering services,116.1494446
Manufacture of soft drinks,Other professional activities,226.4942272
Manufacture of soft drinks,Rental and leasing activities,141.0756669
Manufacture of soft drinks,Business support activities,558.7991451
Manufacture of soft drinks,Public administration,4090.129559
Manufacture of soft drinks,Public education,76.12991218
Manufacture of soft drinks,Private education,97.90419734
Manufacture of soft drinks,Public health activities,1104.730705
Manufacture of soft drinks,Private health activities,1659.722944
Manufacture of soft drinks,Activities of organizations,1469.181816
Manufacture of soft drinks,Artistic and entertainment activities,109.7589811
Manufacture of soft drinks,Other personal services,126.8352618
Manufacture of tobacco products,Cultivation of annual crops,0.04679485
Manufacture of tobacco products,Cultivation of vegetables,0.035490026
Manufacture of tobacco products,Cultivation of grapes,0.066328694
Manufacture of tobacco products,Cultivation of other fruits,0.126195271
Manufacture of tobacco products,Cattle bredding,0.04504485
Manufacture of tobacco products,Pigs breeding,0.006705121
Manufacture of tobacco products,Chicken breeding,0.008221316
Manufacture of tobacco products,Breeding of other animals,0.003158408
Manufacture of tobacco products,Farming services,0.024299733
Manufacture of tobacco products,Forestry,0.061371207
Manufacture of tobacco products,Aquaculture,0.001069486
Manufacture of tobacco products,Extractive fishing,0.012195398
Manufacture of tobacco products,Coal mining,0
Manufacture of tobacco products,Crude oil and gas mining,0.048014301
Manufacture of tobacco products,Copper mining,12.31449532
Manufacture of tobacco products,Iron mining,0.004700348
Manufacture of tobacco products,Mining of other metals,0.026841119
Manufacture of tobacco products,Other mining activities and mining services,0.018333727
Manufacture of tobacco products,Processing and preserving of meat,0.515269726
Manufacture of tobacco products,Processing and preserving of fishmeal and fish oil,0.027805542
Manufacture of tobacco products,Processing and preserving of fish,0.118595173
Manufacture of tobacco products,Processing and preserving of fruits and vegetables,0.024425027
Manufacture of tobacco products,Manufacture of vegetable and animal oils,0.0161662
Manufacture of tobacco products,Manufacture of dairy products,2.994523827
Manufacture of tobacco products,Manufacture of grain mill products,1.11922655
Manufacture of tobacco products,Manufacture of prepared animal feeds,0.631609448
Manufacture of tobacco products,Manufacture of bakery products,1.976868591
Manufacture of tobacco products,Manufacture of macaroni noodles and other related products,0.029137685
Manufacture of tobacco products,Manufacture of other food products,2.486639597
Manufacture of tobacco products,Distilling rectifying and blending of spirits,0.013835488
Manufacture of tobacco products,Manufacture of wines,2.645826351
Manufacture of tobacco products,Manufacture of beers,0.515621372
Manufacture of tobacco products,Manufacture of soft drinks,3.684088213
Manufacture of tobacco products,Manufacture of tobacco products,13206.28805
Manufacture of tobacco products,Manufacture of textile products,0.769614824
Manufacture of tobacco products,Manufacture of wearing apparel,2.302855673
Manufacture of tobacco products,Manufacture of leather products,0.000784831
Manufacture of tobacco products,Manufacture of footwear,0.014443541
Manufacture of tobacco products,Sawmilling and planing of wood,1.881408782
Manufacture of tobacco products,Manufacture of wood products,0.81630264
Manufacture of tobacco products,Manufacture of pulp and paper,1.452376689
Manufacture of tobacco products,Manufacture of containers of paper,0.24569515
Manufacture of tobacco products,Manufacture of other paper products,1.753010011
Manufacture of tobacco products,Printing,1.032883604
Manufacture of tobacco products,Manufacture of refined petroleum products,2.004168101
Manufacture of tobacco products,Manufacture of basic chemicals,1.744794725
Manufacture of tobacco products,Manufacture of paints,0.527012801
Manufacture of tobacco products,Manufacture of pharmaceutical products,3.25754278
Manufacture of tobacco products,Manufacture of soap detergents and toilet products,2.470566592
Manufacture of tobacco products,Manufacture of other chemical products,0.140941045
Manufacture of tobacco products,Manufacture of rubber products,0.076223179
Manufacture of tobacco products,Manufacture of plastic products,0.212058759
Manufacture of tobacco products,Manufacture of glass and glass products,0.476374784
Manufacture of tobacco products,Manufacture of cement,0.436661891
Manufacture of tobacco products,Manufacture of concrete and concrete products,1.413512599
Manufacture of tobacco products,Manufacture of basic iron and steel,0.065621744
Manufacture of tobacco products,Manufacture of other basic metals,0.436032843
Manufacture of tobacco products,Manufacture of fabricated metal products,4.739690871
Manufacture of tobacco products,Manufacture of industrial and domestic machinery and equipment,0.197051894
Manufacture of tobacco products,Manufacture of electric and electronic machinery and equipment,0.823922068
Manufacture of tobacco products,Manufacture of transport equipment,0.563895943
Manufacture of tobacco products,Manufacture of furniture,2.992458649
Manufacture of tobacco products,Repair and installation of machinery and other manufacturing,0.285560198
Manufacture of tobacco products,Electric power generation,0.199373455
Manufacture of tobacco products,Transmission of electric power,0.077761024
Manufacture of tobacco products,Distribution of electric power,1.45474737
Manufacture of tobacco products,Gas and steam manufacture and supply,0.200956817
Manufacture of tobacco products,Water collection treatment and supply,0.32490744
Manufacture of tobacco products,Waste collection and recycling activities,0.487802885
Manufacture of tobacco products,Construction of residential buildings,0.265144838
Manufacture of tobacco products,Construction of non-residential buildings,0.08639472
Manufacture of tobacco products,Civil engineering,1.040891357
Manufacture of tobacco products,Specialized construction activities,0.174271693
Manufacture of tobacco products,Wholesale and retail trade of motor vehicles,1.603520393
Manufacture of tobacco products,Wholesale trade,13.80637014
Manufacture of tobacco products,Retail trade,12.08033251
Manufacture of tobacco products,Accommodation,0.243077572
Manufacture of tobacco products,Food and beverage service activities,1.113591739
Manufacture of tobacco products,Transport via railways,0.15667026
Manufacture of tobacco products,Other passenger land transport,0.244916966
Manufacture of tobacco products,Freight transport by road,0.558444954
Manufacture of tobacco products,Transport via pipeline,0.020566348
Manufacture of tobacco products,Water transport,0.204247708
Manufacture of tobacco products,Air transport,3.980242894
Manufacture of tobacco products,Warehousing,0.039762747
Manufacture of tobacco products,Support activities for land transport,0.573386315
Manufacture of tobacco products,Other support activities for transport,2.829906834
Manufacture of tobacco products,Postal and courier activities,0
Manufacture of tobacco products,Wireless telecommunications,1.912477502
Manufacture of tobacco products,Wired telecommunications,6.174258392
Manufacture of tobacco products,Other telecommunications activities,1.057746829
Manufacture of tobacco products,Information services activities,31.11308453
Manufacture of tobacco products,Publishing activities,0.925096021
Manufacture of tobacco products,Banking monetary intermediation,25.63513824
Manufacture of tobacco products,Insurance activities,5.51870444
Manufacture of tobacco products,Auxiliary financial activities,5.101400246
Manufacture of tobacco products,Real estate activities,0.821511969
Manufacture of tobacco products,Housing services,0
Manufacture of tobacco products,Legal and accounting services,1.134359807
Manufacture of tobacco products,Architectural and engineering services,5.532987686
Manufacture of tobacco products,Other professional activities,16.75098436
Manufacture of tobacco products,Rental and leasing activities,1.862564923
Manufacture of tobacco products,Business support activities,9.69420604
Manufacture of tobacco products,Public administration,1480.20439
Manufacture of tobacco products,Public education,0.229433078
Manufacture of tobacco products,Private education,0.782730622
Manufacture of tobacco products,Public health activities,4.218868182
Manufacture of tobacco products,Private health activities,0.493728516
Manufacture of tobacco products,Activities of organizations,0.162911787
Manufacture of tobacco products,Artistic and entertainment activities,1.56549525
Manufacture of tobacco products,Other personal services,0.308155619
Manufacture of textile products,Cultivation of annual crops,626.494485
Manufacture of textile products,Cultivation of vegetables,656.3891077
Manufacture of textile products,Cultivation of grapes,369.6143867
Manufacture of textile products,Cultivation of other fruits,1169.981557
Manufacture of textile products,Cattle bredding,465.3864798
Manufacture of textile products,Pigs breeding,96.24238591
Manufacture of textile products,Chicken breeding,1107.590435
Manufacture of textile products,Breeding of other animals,34.15789962
Manufacture of textile products,Farming services,195.5601567
Manufacture of textile products,Forestry,360.6856199
Manufacture of textile products,Aquaculture,725.6646557
Manufacture of textile products,Extractive fishing,2342.134722
Manufacture of textile products,Coal mining,16.52956008
Manufacture of textile products,Crude oil and gas mining,7.150760858
Manufacture of textile products,Copper mining,2248.046903
Manufacture of textile products,Iron mining,64.636419
Manufacture of textile products,Mining of other metals,106.9229407
Manufacture of textile products,Other mining activities and mining services,58.84701593
Manufacture of textile products,Processing and preserving of meat,394.0742667
Manufacture of textile products,Processing and preserving of fishmeal and fish oil,46.39334347
Manufacture of textile products,Processing and preserving of fish,332.711878
Manufacture of textile products,Processing and preserving of fruits and vegetables,200.4779071
Manufacture of textile products,Manufacture of vegetable and animal oils,86.36045681
Manufacture of textile products,Manufacture of dairy products,431.1949531
Manufacture of textile products,Manufacture of grain mill products,608.8859791
Manufacture of textile products,Manufacture of prepared animal feeds,468.7398415
Manufacture of textile products,Manufacture of bakery products,336.0663201
Manufacture of textile products,Manufacture of macaroni noodles and other related products,29.77952504
Manufacture of textile products,Manufacture of other food products,416.7906496
Manufacture of textile products,Distilling rectifying and blending of spirits,24.99229291
Manufacture of textile products,Manufacture of wines,269.6192178
Manufacture of textile products,Manufacture of beers,145.8828142
Manufacture of textile products,Manufacture of soft drinks,630.9392574
Manufacture of textile products,Manufacture of tobacco products,31.48755829
Manufacture of textile products,Manufacture of textile products,9033.452314
Manufacture of textile products,Manufacture of wearing apparel,17059.32762
Manufacture of textile products,Manufacture of leather products,5475.004133
Manufacture of textile products,Manufacture of footwear,2457.117564
Manufacture of textile products,Sawmilling and planing of wood,88.12023575
Manufacture of textile products,Manufacture of wood products,135.1195375
Manufacture of textile products,Manufacture of pulp and paper,289.8745442
Manufacture of textile products,Manufacture of containers of paper,120.6351894
Manufacture of textile products,Manufacture of other paper products,3880.607368
Manufacture of textile products,Printing,114.7543099
Manufacture of textile products,Manufacture of refined petroleum products,109.4627852
Manufacture of textile products,Manufacture of basic chemicals,226.287547
Manufacture of textile products,Manufacture of paints,96.50877322
Manufacture of textile products,Manufacture of pharmaceutical products,287.4746922
Manufacture of textile products,Manufacture of soap detergents and toilet products,253.8854997
Manufacture of textile products,Manufacture of other chemical products,135.2127443
Manufacture of textile products,Manufacture of rubber products,3734.075049
Manufacture of textile products,Manufacture of plastic products,615.945258
Manufacture of textile products,Manufacture of glass and glass products,124.619885
Manufacture of textile products,Manufacture of cement,166.0345681
Manufacture of textile products,Manufacture of concrete and concrete products,509.0063468
Manufacture of textile products,Manufacture of basic iron and steel,453.17961
Manufacture of textile products,Manufacture of other basic metals,216.8550363
Manufacture of textile products,Manufacture of fabricated metal products,1053.360981
Manufacture of textile products,Manufacture of industrial and domestic machinery and equipment,510.6043997
Manufacture of textile products,Manufacture of electric and electronic machinery and equipment,207.94362
Manufacture of textile products,Manufacture of transport equipment,480.5016145
Manufacture of textile products,Manufacture of furniture,4443.405827
Manufacture of textile products,Repair and installation of machinery and other manufacturing,1801.181908
Manufacture of textile products,Electric power generation,11698.26286
Manufacture of textile products,Transmission of electric power,6.109967031
Manufacture of textile products,Distribution of electric power,36.27329365
Manufacture of textile products,Gas and steam manufacture and supply,78.09591534
Manufacture of textile products,Water collection treatment and supply,23.07819666
Manufacture of textile products,Waste collection and recycling activities,187.7091758
Manufacture of textile products,Construction of residential buildings,5508.770049
Manufacture of textile products,Construction of non-residential buildings,8690.539868
Manufacture of textile products,Civil engineering,45566.67638
Manufacture of textile products,Specialized construction activities,21618.68641
Manufacture of textile products,Wholesale and retail trade of motor vehicles,1043.289449
Manufacture of textile products,Wholesale trade,3916.242953
Manufacture of textile products,Retail trade,6135.868859
Manufacture of textile products,Accommodation,272.0853543
Manufacture of textile products,Food and beverage service activities,1345.109578
Manufacture of textile products,Transport via railways,21.80397262
Manufacture of textile products,Other passenger land transport,476.9552881
Manufacture of textile products,Freight transport by road,762.0672049
Manufacture of textile products,Transport via pipeline,2.877851008
Manufacture of textile products,Water transport,52.09266106
Manufacture of textile products,Air transport,280.8548157
Manufacture of textile products,Warehousing,29.24833236
Manufacture of textile products,Support activities for land transport,27.87270077
Manufacture of textile products,Other support activities for transport,211.784248
Manufacture of textile products,Postal and courier activities,5.988502585
Manufacture of textile products,Wireless telecommunications,1347.636577
Manufacture of textile products,Wired telecommunications,416.7715407
Manufacture of textile products,Other telecommunications activities,547.3293697
Manufacture of textile products,Information services activities,573.869675
Manufacture of textile products,Publishing activities,262.2230972
Manufacture of textile products,Banking monetary intermediation,600.6967194
Manufacture of textile products,Insurance activities,206.4178653
Manufacture of textile products,Auxiliary financial activities,196.5275406
Manufacture of textile products,Real estate activities,358.1661828
Manufacture of textile products,Housing services,0
Manufacture of textile products,Legal and accounting services,111.6648979
Manufacture of textile products,Architectural and engineering services,997.0117647
Manufacture of textile products,Other professional activities,894.6840283
Manufacture of textile products,Rental and leasing activities,428.5788697
Manufacture of textile products,Business support activities,1770.139075
Manufacture of textile products,Public administration,2054.165561
Manufacture of textile products,Public education,595.5563327
Manufacture of textile products,Private education,237.1921191
Manufacture of textile products,Public health activities,2334.123398
Manufacture of textile products,Private health activities,1572.577005
Manufacture of textile products,Activities of organizations,281.6576123
Manufacture of textile products,Artistic and entertainment activities,308.9862372
Manufacture of textile products,Other personal services,228.3359286
Manufacture of wearing apparel,Cultivation of annual crops,460.5164537
Manufacture of wearing apparel,Cultivation of vegetables,121.8412511
Manufacture of wearing apparel,Cultivation of grapes,254.1900951
Manufacture of wearing apparel,Cultivation of other fruits,631.03446
Manufacture of wearing apparel,Cattle bredding,294.1124667
Manufacture of wearing apparel,Pigs breeding,46.47593669
Manufacture of wearing apparel,Chicken breeding,67.82666966
Manufacture of wearing apparel,Breeding of other animals,29.04001858
Manufacture of wearing apparel,Farming services,168.2835837
Manufacture of wearing apparel,Forestry,1908.212526
Manufacture of wearing apparel,Aquaculture,1806.708651
Manufacture of wearing apparel,Extractive fishing,498.0554389
Manufacture of wearing apparel,Coal mining,14.40462763
Manufacture of wearing apparel,Crude oil and gas mining,40.46056836
Manufacture of wearing apparel,Copper mining,5682.455826
Manufacture of wearing apparel,Iron mining,68.68319955
Manufacture of wearing apparel,Mining of other metals,130.5469686
Manufacture of wearing apparel,Other mining activities and mining services,57.83693093
Manufacture of wearing apparel,Processing and preserving of meat,254.3032264
Manufacture of wearing apparel,Processing and preserving of fishmeal and fish oil,23.92183588
Manufacture of wearing apparel,Processing and preserving of fish,79.36482037
Manufacture of wearing apparel,Processing and preserving of fruits and vegetables,139.5010129
Manufacture of wearing apparel,Manufacture of vegetable and animal oils,63.87208137
Manufacture of wearing apparel,Manufacture of dairy products,254.9800495
Manufacture of wearing apparel,Manufacture of grain mill products,69.04553068
Manufacture of wearing apparel,Manufacture of prepared animal feeds,350.6321473
Manufacture of wearing apparel,Manufacture of bakery products,227.62806
Manufacture of wearing apparel,Manufacture of macaroni noodles and other related products,18.77667987
Manufacture of wearing apparel,Manufacture of other food products,215.2926056
Manufacture of wearing apparel,Distilling rectifying and blending of spirits,13.05284806
Manufacture of wearing apparel,Manufacture of wines,159.8631513
Manufacture of wearing apparel,Manufacture of beers,61.37613142
Manufacture of wearing apparel,Manufacture of soft drinks,195.0791812
Manufacture of wearing apparel,Manufacture of tobacco products,5.607071724
Manufacture of wearing apparel,Manufacture of textile products,213.2895484
Manufacture of wearing apparel,Manufacture of wearing apparel,6469.500332
Manufacture of wearing apparel,Manufacture of leather products,114.5250283
Manufacture of wearing apparel,Manufacture of footwear,341.9491689
Manufacture of wearing apparel,Sawmilling and planing of wood,39.5212778
Manufacture of wearing apparel,Manufacture of wood products,91.84152907
Manufacture of wearing apparel,Manufacture of pulp and paper,231.0432078
Manufacture of wearing apparel,Manufacture of containers of paper,80.80840354
Manufacture of wearing apparel,Manufacture of other paper products,233.4076621
Manufacture of wearing apparel,Printing,77.21546935
Manufacture of wearing apparel,Manufacture of refined petroleum products,62.34024403
Manufacture of wearing apparel,Manufacture of basic chemicals,151.8806241
Manufacture of wearing apparel,Manufacture of paints,68.50171373
Manufacture of wearing apparel,Manufacture of pharmaceutical products,144.5500643
Manufacture of wearing apparel,Manufacture of soap detergents and toilet products,160.7236952
Manufacture of wearing apparel,Manufacture of other chemical products,98.57763749
Manufacture of wearing apparel,Manufacture of rubber products,92.04557788
Manufacture of wearing apparel,Manufacture of plastic products,462.5289698
Manufacture of wearing apparel,Manufacture of glass and glass products,107.602039
Manufacture of wearing apparel,Manufacture of cement,93.59243451
Manufacture of wearing apparel,Manufacture of concrete and concrete products,280.8895084
Manufacture of wearing apparel,Manufacture of basic iron and steel,398.39507
Manufacture of wearing apparel,Manufacture of other basic metals,193.1414407
Manufacture of wearing apparel,Manufacture of fabricated metal products,487.2722811
Manufacture of wearing apparel,Manufacture of industrial and domestic machinery and equipment,421.6951161
Manufacture of wearing apparel,Manufacture of electric and electronic machinery and equipment,166.6494485
Manufacture of wearing apparel,Manufacture of transport equipment,201.4645792
Manufacture of wearing apparel,Manufacture of furniture,369.9065026
Manufacture of wearing apparel,Repair and installation of machinery and other manufacturing,1811.038461
Manufacture of wearing apparel,Electric power generation,391.9700135
Manufacture of wearing apparel,Transmission of electric power,10.30036472
Manufacture of wearing apparel,Distribution of electric power,50.5604947
Manufacture of wearing apparel,Gas and steam manufacture and supply,10.34878035
Manufacture of wearing apparel,Water collection treatment and supply,85.28133776
Manufacture of wearing apparel,Waste collection and recycling activities,2094.339211
Manufacture of wearing apparel,Construction of residential buildings,1170.513366
Manufacture of wearing apparel,Construction of non-residential buildings,699.6746423
Manufacture of wearing apparel,Civil engineering,3194.617744
Manufacture of wearing apparel,Specialized construction activities,1990.291829
Manufacture of wearing apparel,Wholesale and retail trade of motor vehicles,2392.073365
Manufacture of wearing apparel,Wholesale trade,9153.309577
Manufacture of wearing apparel,Retail trade,4746.312074
Manufacture of wearing apparel,Accommodation,1700.372534
Manufacture of wearing apparel,Food and beverage service activities,7367.776146
Manufacture of wearing apparel,Transport via railways,21.39246618
Manufacture of wearing apparel,Other passenger land transport,663.9924761
Manufacture of wearing apparel,Freight transport by road,577.2528203
Manufacture of wearing apparel,Transport via pipeline,5.389091833
Manufacture of wearing apparel,Water transport,48.555251
Manufacture of wearing apparel,Air transport,169.9351545
Manufacture of wearing apparel,Warehousing,3.572290744
Manufacture of wearing apparel,Support activities for land transport,11.79331776
Manufacture of wearing apparel,Other support activities for transport,197.8187078
Manufacture of wearing apparel,Postal and courier activities,4.748260679
Manufacture of wearing apparel,Wireless telecommunications,1143.516604
Manufacture of wearing apparel,Wired telecommunications,377.9713751
Manufacture of wearing apparel,Other telecommunications activities,433.1805161
Manufacture of wearing apparel,Information services activities,472.6933903
Manufacture of wearing apparel,Publishing activities,474.6959374
Manufacture of wearing apparel,Banking monetary intermediation,410.140104
Manufacture of wearing apparel,Insurance activities,105.7387225
Manufacture of wearing apparel,Auxiliary financial activities,125.3507229
Manufacture of wearing apparel,Real estate activities,1238.381762
Manufacture of wearing apparel,Housing services,0
Manufacture of wearing apparel,Legal and accounting services,388.163859
Manufacture of wearing apparel,Architectural and engineering services,5878.648525
Manufacture of wearing apparel,Other professional activities,2832.621393
Manufacture of wearing apparel,Rental and leasing activities,2287.44911
Manufacture of wearing apparel,Business support activities,11612.15474
Manufacture of wearing apparel,Public administration,26020.95117
Manufacture of wearing apparel,Public education,7459.090661
Manufacture of wearing apparel,Private education,291.4609725
Manufacture of wearing apparel,Public health activities,6776.721992
Manufacture of wearing apparel,Private health activities,2567.404099
Manufacture of wearing apparel,Activities of organizations,1771.176197
Manufacture of wearing apparel,Artistic and entertainment activities,1925.667105
Manufacture of wearing apparel,Other personal services,529.018397
Manufacture of leather products,Cultivation of annual crops,154.6231317
Manufacture of leather products,Cultivation of vegetables,66.23374585
Manufacture of leather products,Cultivation of grapes,49.38006647
Manufacture of leather products,Cultivation of other fruits,73.24224798
Manufacture of leather products,Cattle bredding,145.7635035
Manufacture of leather products,Pigs breeding,28.58352809
Manufacture of leather products,Chicken breeding,31.25191917
Manufacture of leather products,Breeding of other animals,3.497583305
Manufacture of leather products,Farming services,20.26521653
Manufacture of leather products,Forestry,14.12465409
Manufacture of leather products,Aquaculture,22.26720008
Manufacture of leather products,Extractive fishing,9.709683333
Manufacture of leather products,Coal mining,1.702242871
Manufacture of leather products,Crude oil and gas mining,0.404474097
Manufacture of leather products,Copper mining,169.3803928
Manufacture of leather products,Iron mining,5.540715231
Manufacture of leather products,Mining of other metals,9.821905831
Manufacture of leather products,Other mining activities and mining services,5.103692655
Manufacture of leather products,Processing and preserving of meat,30.516662
Manufacture of leather products,Processing and preserving of fishmeal and fish oil,2.723390023
Manufacture of leather products,Processing and preserving of fish,8.66677768
Manufacture of leather products,Processing and preserving of fruits and vegetables,16.54030995
Manufacture of leather products,Manufacture of vegetable and animal oils,7.69727397
Manufacture of leather products,Manufacture of dairy products,30.01834025
Manufacture of leather products,Manufacture of grain mill products,7.056686471
Manufacture of leather products,Manufacture of prepared animal feeds,41.75046013
Manufacture of leather products,Manufacture of bakery products,26.49045926
Manufacture of leather products,Manufacture of macaroni noodles and other related products,2.253617479
Manufacture of leather products,Manufacture of other food products,24.76501988
Manufacture of leather products,Distilling rectifying and blending of spirits,1.581360049
Manufacture of leather products,Manufacture of wines,18.22056326
Manufacture of leather products,Manufacture of beers,7.168037182
Manufacture of leather products,Manufacture of soft drinks,22.07152148
Manufacture of leather products,Manufacture of tobacco products,0.684986019
Manufacture of leather products,Manufacture of textile products,56.4200442
Manufacture of leather products,Manufacture of wearing apparel,2374.126964
Manufacture of leather products,Manufacture of leather products,2111.922684
Manufacture of leather products,Manufacture of footwear,19257.9825
Manufacture of leather products,Sawmilling and planing of wood,4.393595901
Manufacture of leather products,Manufacture of wood products,38.93905191
Manufacture of leather products,Manufacture of pulp and paper,25.31401848
Manufacture of leather products,Manufacture of containers of paper,8.582625378
Manufacture of leather products,Manufacture of other paper products,21.69838297
Manufacture of leather products,Printing,66.41286426
Manufacture of leather products,Manufacture of refined petroleum products,7.104881998
Manufacture of leather products,Manufacture of basic chemicals,17.25860669
Manufacture of leather products,Manufacture of paints,8.149331204
Manufacture of leather products,Manufacture of pharmaceutical products,16.31757744
Manufacture of leather products,Manufacture of soap detergents and toilet products,18.34212184
Manufacture of leather products,Manufacture of other chemical products,11.45015314
Manufacture of leather products,Manufacture of rubber products,24.2021324
Manufacture of leather products,Manufacture of plastic products,52.27329977
Manufacture of leather products,Manufacture of glass and glass products,12.22284385
Manufacture of leather products,Manufacture of cement,10.34488863
Manufacture of leather products,Manufacture of concrete and concrete products,33.28962992
Manufacture of leather products,Manufacture of basic iron and steel,44.43353214
Manufacture of leather products,Manufacture of other basic metals,21.17908743
Manufacture of leather products,Manufacture of fabricated metal products,196.70263
Manufacture of leather products,Manufacture of industrial and domestic machinery and equipment,112.6997444
Manufacture of leather products,Manufacture of electric and electronic machinery and equipment,77.42477012
Manufacture of leather products,Manufacture of transport equipment,33.87063391
Manufacture of leather products,Manufacture of furniture,356.1815297
Manufacture of leather products,Repair and installation of machinery and other manufacturing,336.0860466
Manufacture of leather products,Electric power generation,68.17500551
Manufacture of leather products,Transmission of electric power,0.133143351
Manufacture of leather products,Distribution of electric power,1.780437258
Manufacture of leather products,Gas and steam manufacture and supply,0.744507644
Manufacture of leather products,Water collection treatment and supply,1.415985995
Manufacture of leather products,Waste collection and recycling activities,2.976289835
Manufacture of leather products,Construction of residential buildings,98.93614888
Manufacture of leather products,Construction of non-residential buildings,70.39413785
Manufacture of leather products,Civil engineering,181.2372984
Manufacture of leather products,Specialized construction activities,193.8183921
Manufacture of leather products,Wholesale and retail trade of motor vehicles,85.31812743
Manufacture of leather products,Wholesale trade,255.3679307
Manufacture of leather products,Retail trade,3309.209744
Manufacture of leather products,Accommodation,17.4470356
Manufacture of leather products,Food and beverage service activities,92.05198124
Manufacture of leather products,Transport via railways,1.340349673
Manufacture of leather products,Other passenger land transport,44.04058943
Manufacture of leather products,Freight transport by road,58.43470109
Manufacture of leather products,Transport via pipeline,0.032301162
Manufacture of leather products,Water transport,4.754104468
Manufacture of leather products,Air transport,18.21063994
Manufacture of leather products,Warehousing,0.268599233
Manufacture of leather products,Support activities for land transport,1.050316565
Manufacture of leather products,Other support activities for transport,20.1179429
Manufacture of leather products,Postal and courier activities,0.427758401
Manufacture of leather products,Wireless telecommunications,134.2302537
Manufacture of leather products,Wired telecommunications,39.74714699
Manufacture of leather products,Other telecommunications activities,46.07882031
Manufacture of leather products,Information services activities,49.34109135
Manufacture of leather products,Publishing activities,16.97597313
Manufacture of leather products,Banking monetary intermediation,120.1136361
Manufacture of leather products,Insurance activities,239.6393732
Manufacture of leather products,Auxiliary financial activities,66.69699436
Manufacture of leather products,Real estate activities,12.53510679
Manufacture of leather products,Housing services,0
Manufacture of leather products,Legal and accounting services,2.229354143
Manufacture of leather products,Architectural and engineering services,19.63448155
Manufacture of leather products,Other professional activities,41.31173059
Manufacture of leather products,Rental and leasing activities,21.98759923
Manufacture of leather products,Business support activities,88.09055422
Manufacture of leather products,Public administration,2933.030758
Manufacture of leather products,Public education,104.3184075
Manufacture of leather products,Private education,31.44733155
Manufacture of leather products,Public health activities,215.8609889
Manufacture of leather products,Private health activities,284.5373748
Manufacture of leather products,Activities of organizations,133.1577522
Manufacture of leather products,Artistic and entertainment activities,17.18170888
Manufacture of leather products,Other personal services,19.27019123
Manufacture of footwear,Cultivation of annual crops,205.5032794
Manufacture of footwear,Cultivation of vegetables,192.1555284
Manufacture of footwear,Cultivation of grapes,229.2215089
Manufacture of footwear,Cultivation of other fruits,267.4701877
Manufacture of footwear,Cattle bredding,123.5112796
Manufacture of footwear,Pigs breeding,63.55551913
Manufacture of footwear,Chicken breeding,54.85955519
Manufacture of footwear,Breeding of other animals,84.79357123
Manufacture of footwear,Farming services,79.94267401
Manufacture of footwear,Forestry,59.8477564
Manufacture of footwear,Aquaculture,79.61115311
Manufacture of footwear,Extractive fishing,50.14663456
Manufacture of footwear,Coal mining,5.99758616
Manufacture of footwear,Crude oil and gas mining,1.295918481
Manufacture of footwear,Copper mining,563.6160743
Manufacture of footwear,Iron mining,19.50748367
Manufacture of footwear,Mining of other metals,34.53828426
Manufacture of footwear,Other mining activities and mining services,17.96943982
Manufacture of footwear,Processing and preserving of meat,107.9258836
Manufacture of footwear,Processing and preserving of fishmeal and fish oil,9.648780723
Manufacture of footwear,Processing and preserving of fish,31.49534876
Manufacture of footwear,Processing and preserving of fruits and vegetables,59.00776044
Manufacture of footwear,Manufacture of vegetable and animal oils,27.36428015
Manufacture of footwear,Manufacture of dairy products,99.56094885
Manufacture of footwear,Manufacture of grain mill products,22.33039949
Manufacture of footwear,Manufacture of prepared animal feeds,146.909125
Manufacture of footwear,Manufacture of bakery products,88.1631064
Manufacture of footwear,Manufacture of macaroni noodles and other related products,8.108764085
Manufacture of footwear,Manufacture of other food products,83.82473846
Manufacture of footwear,Distilling rectifying and blending of spirits,5.658655496
Manufacture of footwear,Manufacture of wines,58.03812876
Manufacture of footwear,Manufacture of beers,25.01191961
Manufacture of footwear,Manufacture of soft drinks,73.01096801
Manufacture of footwear,Manufacture of tobacco products,3.054361727
Manufacture of footwear,Manufacture of textile products,41.45302936
Manufacture of footwear,Manufacture of wearing apparel,113.3871913
Manufacture of footwear,Manufacture of leather products,5.38600457
Manufacture of footwear,Manufacture of footwear,30.98128684
Manufacture of footwear,Sawmilling and planing of wood,10.65016577
Manufacture of footwear,Manufacture of wood products,31.14037728
Manufacture of footwear,Manufacture of pulp and paper,85.51720138
Manufacture of footwear,Manufacture of containers of paper,30.30338587
Manufacture of footwear,Manufacture of other paper products,72.4704291
Manufacture of footwear,Printing,26.94037624
Manufacture of footwear,Manufacture of refined petroleum products,20.10676857
Manufacture of footwear,Manufacture of basic chemicals,57.11210865
Manufacture of footwear,Manufacture of paints,27.59241826
Manufacture of footwear,Manufacture of pharmaceutical products,50.60308839
Manufacture of footwear,Manufacture of soap detergents and toilet products,59.32204794
Manufacture of footwear,Manufacture of other chemical products,40.31288245
Manufacture of footwear,Manufacture of rubber products,19.16056603
Manufacture of footwear,Manufacture of plastic products,186.6964112
Manufacture of footwear,Manufacture of glass and glass products,41.75732407
Manufacture of footwear,Manufacture of cement,35.45983663
Manufacture of footwear,Manufacture of concrete and concrete products,113.9581811
Manufacture of footwear,Manufacture of basic iron and steel,156.5075893
Manufacture of footwear,Manufacture of other basic metals,73.43522986
Manufacture of footwear,Manufacture of fabricated metal products,194.7950338
Manufacture of footwear,Manufacture of industrial and domestic machinery and equipment,172.5349517
Manufacture of footwear,Manufacture of electric and electronic machinery and equipment,65.77466131
Manufacture of footwear,Manufacture of transport equipment,41.71886762
Manufacture of footwear,Manufacture of furniture,124.9731678
Manufacture of footwear,Repair and installation of machinery and other manufacturing,102.6933783
Manufacture of footwear,Electric power generation,97.50375746
Manufacture of footwear,Transmission of electric power,0.249965768
Manufacture of footwear,Distribution of electric power,2.141702941
Manufacture of footwear,Gas and steam manufacture and supply,2.934175905
Manufacture of footwear,Water collection treatment and supply,4.120445855
Manufacture of footwear,Waste collection and recycling activities,9.487668468
Manufacture of footwear,Construction of residential buildings,353.548017
Manufacture of footwear,Construction of non-residential buildings,254.7933119
Manufacture of footwear,Civil engineering,682.5310704
Manufacture of footwear,Specialized construction activities,691.7807831
Manufacture of footwear,Wholesale and retail trade of motor vehicles,298.3533144
Manufacture of footwear,Wholesale trade,867.1897406
Manufacture of footwear,Retail trade,1610.706335
Manufacture of footwear,Accommodation,61.1135645
Manufacture of footwear,Food and beverage service activities,323.2206757
Manufacture of footwear,Transport via railways,4.323932091
Manufacture of footwear,Other passenger land transport,154.6029023
Manufacture of footwear,Freight transport by road,205.297822
Manufacture of footwear,Transport via pipeline,0.055319574
Manufacture of footwear,Water transport,16.19162628
Manufacture of footwear,Air transport,53.10577339
Manufacture of footwear,Warehousing,0.960432441
Manufacture of footwear,Support activities for land transport,2.27655804
Manufacture of footwear,Other support activities for transport,62.86059169
Manufacture of footwear,Postal and courier activities,1.523453133
Manufacture of footwear,Wireless telecommunications,468.0102877
Manufacture of footwear,Wired telecommunications,122.5511186
Manufacture of footwear,Other telecommunications activities,160.0810077
Manufacture of footwear,Information services activities,86.03016244
Manufacture of footwear,Publishing activities,57.78476714
Manufacture of footwear,Banking monetary intermediation,80.35024518
Manufacture of footwear,Insurance activities,24.46251923
Manufacture of footwear,Auxiliary financial activities,31.60395997
Manufacture of footwear,Real estate activities,43.30917637
Manufacture of footwear,Housing services,0
Manufacture of footwear,Legal and accounting services,5.485398489
Manufacture of footwear,Architectural and engineering services,55.97077749
Manufacture of footwear,Other professional activities,101.5448148
Manufacture of footwear,Rental and leasing activities,73.06765279
Manufacture of footwear,Business support activities,288.5339971
Manufacture of footwear,Public administration,101.8122943
Manufacture of footwear,Public education,1353.998264
Manufacture of footwear,Private education,39.51399279
Manufacture of footwear,Public health activities,1102.176424
Manufacture of footwear,Private health activities,453.4522587
Manufacture of footwear,Activities of organizations,60.61605507
Manufacture of footwear,Artistic and entertainment activities,56.73807067
Manufacture of footwear,Other personal services,67.24783574
Sawmilling and planing of wood,Cultivation of annual crops,2845.791365
Sawmilling and planing of wood,Cultivation of vegetables,2069.985058
Sawmilling and planing of wood,Cultivation of grapes,3314.668202
Sawmilling and planing of wood,Cultivation of other fruits,10742.94988
Sawmilling and planing of wood,Cattle bredding,2343.608915
Sawmilling and planing of wood,Pigs breeding,1018.979547
Sawmilling and planing of wood,Chicken breeding,2017.262988
Sawmilling and planing of wood,Breeding of other animals,246.5907235
Sawmilling and planing of wood,Farming services,92.73002202
Sawmilling and planing of wood,Forestry,2062.010986
Sawmilling and planing of wood,Aquaculture,92.53640524
Sawmilling and planing of wood,Extractive fishing,69.08930152
Sawmilling and planing of wood,Coal mining,352.3811469
Sawmilling and planing of wood,Crude oil and gas mining,1.982930098
Sawmilling and planing of wood,Copper mining,1146.453303
Sawmilling and planing of wood,Iron mining,29.68339889
Sawmilling and planing of wood,Mining of other metals,92.49458651
Sawmilling and planing of wood,Other mining activities and mining services,978.3314263
Sawmilling and planing of wood,Processing and preserving of meat,218.7432946
Sawmilling and planing of wood,Processing and preserving of fishmeal and fish oil,19.27139895
Sawmilling and planing of wood,Processing and preserving of fish,120.6583305
Sawmilling and planing of wood,Processing and preserving of fruits and vegetables,102.2710657
Sawmilling and planing of wood,Manufacture of vegetable and animal oils,38.55917071
Sawmilling and planing of wood,Manufacture of dairy products,180.1014376
Sawmilling and planing of wood,Manufacture of grain mill products,62.81269906
Sawmilling and planing of wood,Manufacture of prepared animal feeds,200.8487493
Sawmilling and planing of wood,Manufacture of bakery products,160.7087374
Sawmilling and planing of wood,Manufacture of macaroni noodles and other related products,9.771129929
Sawmilling and planing of wood,Manufacture of other food products,155.8157022
Sawmilling and planing of wood,Distilling rectifying and blending of spirits,20.86610443
Sawmilling and planing of wood,Manufacture of wines,1619.815458
Sawmilling and planing of wood,Manufacture of beers,79.96723144
Sawmilling and planing of wood,Manufacture of soft drinks,204.8970943
Sawmilling and planing of wood,Manufacture of tobacco products,13.94666104
Sawmilling and planing of wood,Manufacture of textile products,115.851362
Sawmilling and planing of wood,Manufacture of wearing apparel,172.7409453
Sawmilling and planing of wood,Manufacture of leather products,4.726418773
Sawmilling and planing of wood,Manufacture of footwear,21.80168753
Sawmilling and planing of wood,Sawmilling and planing of wood,165994.9162
Sawmilling and planing of wood,Manufacture of wood products,105144.0878
Sawmilling and planing of wood,Manufacture of pulp and paper,16962.51625
Sawmilling and planing of wood,Manufacture of containers of paper,138.6184907
Sawmilling and planing of wood,Manufacture of other paper products,132.7542896
Sawmilling and planing of wood,Printing,64.60372154
Sawmilling and planing of wood,Manufacture of refined petroleum products,90.88612701
Sawmilling and planing of wood,Manufacture of basic chemicals,255.8898904
Sawmilling and planing of wood,Manufacture of paints,47.88013229
Sawmilling and planing of wood,Manufacture of pharmaceutical products,195.4861962
Sawmilling and planing of wood,Manufacture of soap detergents and toilet products,127.1741338
Sawmilling and planing of wood,Manufacture of other chemical products,973.6383985
Sawmilling and planing of wood,Manufacture of rubber products,22.55727217
Sawmilling and planing of wood,Manufacture of plastic products,553.7374408
Sawmilling and planing of wood,Manufacture of glass and glass products,57.90577586
Sawmilling and planing of wood,Manufacture of cement,81.8994672
Sawmilling and planing of wood,Manufacture of concrete and concrete products,414.5853955
Sawmilling and planing of wood,Manufacture of basic iron and steel,203.0439895
Sawmilling and planing of wood,Manufacture of other basic metals,135.6769848
Sawmilling and planing of wood,Manufacture of fabricated metal products,2326.44239
Sawmilling and planing of wood,Manufacture of industrial and domestic machinery and equipment,515.1685477
Sawmilling and planing of wood,Manufacture of electric and electronic machinery and equipment,135.5527434
Sawmilling and planing of wood,Manufacture of transport equipment,301.0915126
Sawmilling and planing of wood,Manufacture of furniture,20313.52876
Sawmilling and planing of wood,Repair and installation of machinery and other manufacturing,2400.783669
Sawmilling and planing of wood,Electric power generation,109.4620753
Sawmilling and planing of wood,Transmission of electric power,1.099018561
Sawmilling and planing of wood,Distribution of electric power,433.8209181
Sawmilling and planing of wood,Gas and steam manufacture and supply,96.22869926
Sawmilling and planing of wood,Water collection treatment and supply,11.37036531
Sawmilling and planing of wood,Waste collection and recycling activities,20.05131553
Sawmilling and planing of wood,Construction of residential buildings,68806.09314
Sawmilling and planing of wood,Construction of non-residential buildings,14824.09233
Sawmilling and planing of wood,Civil engineering,59332.61574
Sawmilling and planing of wood,Specialized construction activities,100557.8399
Sawmilling and planing of wood,Wholesale and retail trade of motor vehicles,419.8108698
Sawmilling and planing of wood,Wholesale trade,1722.703195
Sawmilling and planing of wood,Retail trade,2214.794704
Sawmilling and planing of wood,Accommodation,74.16618571
Sawmilling and planing of wood,Food and beverage service activities,386.5862884
Sawmilling and planing of wood,Transport via railways,10.89017984
Sawmilling and planing of wood,Other passenger land transport,177.9896575
Sawmilling and planing of wood,Freight transport by road,402.4178264
Sawmilling and planing of wood,Transport via pipeline,0.20726192
Sawmilling and planing of wood,Water transport,23.16649395
Sawmilling and planing of wood,Air transport,112.0762482
Sawmilling and planing of wood,Warehousing,19.97454192
Sawmilling and planing of wood,Support activities for land transport,27.73608172
Sawmilling and planing of wood,Other support activities for transport,92.38671826
Sawmilling and planing of wood,Postal and courier activities,9.576393342
Sawmilling and planing of wood,Wireless telecommunications,573.1468551
Sawmilling and planing of wood,Wired telecommunications,188.1688318
Sawmilling and planing of wood,Other telecommunications activities,255.1557351
Sawmilling and planing of wood,Information services activities,415.9718813
Sawmilling and planing of wood,Publishing activities,125.9922381
Sawmilling and planing of wood,Banking monetary intermediation,476.0847013
Sawmilling and planing of wood,Insurance activities,187.3653573
Sawmilling and planing of wood,Auxiliary financial activities,144.6881762
Sawmilling and planing of wood,Real estate activities,164.5571544
Sawmilling and planing of wood,Housing services,0
Sawmilling and planing of wood,Legal and accounting services,94.55533169
Sawmilling and planing of wood,Architectural and engineering services,526.7676389
Sawmilling and planing of wood,Other professional activities,562.575859
Sawmilling and planing of wood,Rental and leasing activities,150.3972344
Sawmilling and planing of wood,Business support activities,634.569177
Sawmilling and planing of wood,Public administration,641.6590589
Sawmilling and planing of wood,Public education,93.6339288
Sawmilling and planing of wood,Private education,161.002477
Sawmilling and planing of wood,Public health activities,773.3154298
Sawmilling and planing of wood,Private health activities,594.9770052
Sawmilling and planing of wood,Activities of organizations,84.60949495
Sawmilling and planing of wood,Artistic and entertainment activities,101.9741428
Sawmilling and planing of wood,Other personal services,104.3889377
Manufacture of wood products,Cultivation of annual crops,368.2281607
Manufacture of wood products,Cultivation of vegetables,707.2887305
Manufacture of wood products,Cultivation of grapes,389.2060661
Manufacture of wood products,Cultivation of other fruits,1229.817223
Manufacture of wood products,Cattle bredding,291.0393484
Manufacture of wood products,Pigs breeding,115.6360484
Manufacture of wood products,Chicken breeding,234.0433477
Manufacture of wood products,Breeding of other animals,30.19084995
Manufacture of wood products,Farming services,206.1495629
Manufacture of wood products,Forestry,765.3006348
Manufacture of wood products,Aquaculture,33.41386251
Manufacture of wood products,Extractive fishing,658.0290305
Manufacture of wood products,Coal mining,40.56002588
Manufacture of wood products,Crude oil and gas mining,1.946559942
Manufacture of wood products,Copper mining,10533.37477
Manufacture of wood products,Iron mining,304.2003692
Manufacture of wood products,Mining of other metals,1165.970675
Manufacture of wood products,Other mining activities and mining services,303.7696217
Manufacture of wood products,Processing and preserving of meat,292.8745146
Manufacture of wood products,Processing and preserving of fishmeal and fish oil,91.86360297
Manufacture of wood products,Processing and preserving of fish,577.3058917
Manufacture of wood products,Processing and preserving of fruits and vegetables,130.1467018
Manufacture of wood products,Manufacture of vegetable and animal oils,12.13113844
Manufacture of wood products,Manufacture of dairy products,220.0855388
Manufacture of wood products,Manufacture of grain mill products,73.70980446
Manufacture of wood products,Manufacture of prepared animal feeds,77.64477364
Manufacture of wood products,Manufacture of bakery products,387.1141646
Manufacture of wood products,Manufacture of macaroni noodles and other related products,9.779725899
Manufacture of wood products,Manufacture of other food products,198.5758254
Manufacture of wood products,Distilling rectifying and blending of spirits,5.3349395
Manufacture of wood products,Manufacture of wines,24495.97064
Manufacture of wood products,Manufacture of beers,61.01705703
Manufacture of wood products,Manufacture of soft drinks,123.9855038
Manufacture of wood products,Manufacture of tobacco products,1.289559087
Manufacture of wood products,Manufacture of textile products,1034.821163
Manufacture of wood products,Manufacture of wearing apparel,103.2820316
Manufacture of wood products,Manufacture of leather products,6.189158755
Manufacture of wood products,Manufacture of footwear,130.0716753
Manufacture of wood products,Sawmilling and planing of wood,32151.74939
Manufacture of wood products,Manufacture of wood products,31655.74372
Manufacture of wood products,Manufacture of pulp and paper,4008.083226
Manufacture of wood products,Manufacture of containers of paper,1682.079782
Manufacture of wood products,Manufacture of other paper products,208.1668673
Manufacture of wood products,Printing,58.62790778
Manufacture of wood products,Manufacture of refined petroleum products,325.5293634
Manufacture of wood products,Manufacture of basic chemicals,686.0865556
Manufacture of wood products,Manufacture of paints,37.61531304
Manufacture of wood products,Manufacture of pharmaceutical products,208.2876254
Manufacture of wood products,Manufacture of soap detergents and toilet products,83.48287246
Manufacture of wood products,Manufacture of other chemical products,123.0462694
Manufacture of wood products,Manufacture of rubber products,30.167793
Manufacture of wood products,Manufacture of plastic products,848.55806
Manufacture of wood products,Manufacture of glass and glass products,129.9158494
Manufacture of wood products,Manufacture of cement,256.8363716
Manufacture of wood products,Manufacture of concrete and concrete products,4352.479013
Manufacture of wood products,Manufacture of basic iron and steel,207.760058
Manufacture of wood products,Manufacture of other basic metals,979.6703441
Manufacture of wood products,Manufacture of fabricated metal products,1067.79728
Manufacture of wood products,Manufacture of industrial and domestic machinery and equipment,782.3639889
Manufacture of wood products,Manufacture of electric and electronic machinery and equipment,181.7196494
Manufacture of wood products,Manufacture of transport equipment,208.5170606
Manufacture of wood products,Manufacture of furniture,59486.02111
Manufacture of wood products,Repair and installation of machinery and other manufacturing,885.0256122
Manufacture of wood products,Electric power generation,49.38655875
Manufacture of wood products,Transmission of electric power,1.948879372
Manufacture of wood products,Distribution of electric power,15181.54683
Manufacture of wood products,Gas and steam manufacture and supply,96.69895528
Manufacture of wood products,Water collection treatment and supply,9.559800821
Manufacture of wood products,Waste collection and recycling activities,15.39943669
Manufacture of wood products,Construction of residential buildings,52581.22631
Manufacture of wood products,Construction of non-residential buildings,20758.44498
Manufacture of wood products,Civil engineering,104534.3319
Manufacture of wood products,Specialized construction activities,148717.8251
Manufacture of wood products,Wholesale and retail trade of motor vehicles,288.7408553
Manufacture of wood products,Wholesale trade,1219.648771
Manufacture of wood products,Retail trade,1673.491143
Manufacture of wood products,Accommodation,31.60134023
Manufacture of wood products,Food and beverage service activities,162.6029698
Manufacture of wood products,Transport via railways,5.632777429
Manufacture of wood products,Other passenger land transport,71.61920812
Manufacture of wood products,Freight transport by road,118.5388398
Manufacture of wood products,Transport via pipeline,0.509834049
Manufacture of wood products,Water transport,11.72678227
Manufacture of wood products,Air transport,125.675579
Manufacture of wood products,Warehousing,1.577533083
Manufacture of wood products,Support activities for land transport,14.83968472
Manufacture of wood products,Other support activities for transport,99.4480084
Manufacture of wood products,Postal and courier activities,0.669045497
Manufacture of wood products,Wireless telecommunications,243.7980183
Manufacture of wood products,Wired telecommunications,197.6382803
Manufacture of wood products,Other telecommunications activities,93.92802796
Manufacture of wood products,Information services activities,982.002673
Manufacture of wood products,Publishing activities,72.89823963
Manufacture of wood products,Banking monetary intermediation,1022.478771
Manufacture of wood products,Insurance activities,166.112642
Manufacture of wood products,Auxiliary financial activities,202.2053442
Manufacture of wood products,Real estate activities,42.04937999
Manufacture of wood products,Housing services,0
Manufacture of wood products,Legal and accounting services,30.67198706
Manufacture of wood products,Architectural and engineering services,185.5309199
Manufacture of wood products,Other professional activities,453.4230831
Manufacture of wood products,Rental and leasing activities,78.05742184
Manufacture of wood products,Business support activities,354.3319975
Manufacture of wood products,Public administration,1525.037304
Manufacture of wood products,Public education,22.15624062
Manufacture of wood products,Private education,37.07767831
Manufacture of wood products,Public health activities,333.7708118
Manufacture of wood products,Private health activities,747.8181172
Manufacture of wood products,Activities of organizations,375.7274852
Manufacture of wood products,Artistic and entertainment activities,61.32267926
Manufacture of wood products,Other personal services,572.1326334
Manufacture of pulp and paper,Cultivation of annual crops,234.1282624
Manufacture of pulp and paper,Cultivation of vegetables,106.1384128
Manufacture of pulp and paper,Cultivation of grapes,396.6981999
Manufacture of pulp and paper,Cultivation of other fruits,930.9831041
Manufacture of pulp and paper,Cattle bredding,1656.069712
Manufacture of pulp and paper,Pigs breeding,815.3726764
Manufacture of pulp and paper,Chicken breeding,1625.522469
Manufacture of pulp and paper,Breeding of other animals,170.4724513
Manufacture of pulp and paper,Farming services,214.7531677
Manufacture of pulp and paper,Forestry,180.1349119
Manufacture of pulp and paper,Aquaculture,81.92613417
Manufacture of pulp and paper,Extractive fishing,0.437073968
Manufacture of pulp and paper,Coal mining,14.08466284
Manufacture of pulp and paper,Crude oil and gas mining,91.26304919
Manufacture of pulp and paper,Copper mining,160629.106
Manufacture of pulp and paper,Iron mining,4882.169801
Manufacture of pulp and paper,Mining of other metals,3160.971406
Manufacture of pulp and paper,Other mining activities and mining services,3022.284797
Manufacture of pulp and paper,Processing and preserving of meat,4370.669773
Manufacture of pulp and paper,Processing and preserving of fishmeal and fish oil,576.2596378
Manufacture of pulp and paper,Processing and preserving of fish,5027.078378
Manufacture of pulp and paper,Processing and preserving of fruits and vegetables,2164.840804
Manufacture of pulp and paper,Manufacture of vegetable and animal oils,118.1772061
Manufacture of pulp and paper,Manufacture of dairy products,5389.284358
Manufacture of pulp and paper,Manufacture of grain mill products,730.216317
Manufacture of pulp and paper,Manufacture of prepared animal feeds,509.413658
Manufacture of pulp and paper,Manufacture of bakery products,803.2770968
Manufacture of pulp and paper,Manufacture of macaroni noodles and other related products,271.8515938
Manufacture of pulp and paper,Manufacture of other food products,2640.157428
Manufacture of pulp and paper,Distilling rectifying and blending of spirits,854.9703204
Manufacture of pulp and paper,Manufacture of wines,633.245373
Manufacture of pulp and paper,Manufacture of beers,4394.555755
Manufacture of pulp and paper,Manufacture of soft drinks,3536.903728
Manufacture of pulp and paper,Manufacture of tobacco products,3069.316903
Manufacture of pulp and paper,Manufacture of textile products,580.0404377
Manufacture of pulp and paper,Manufacture of wearing apparel,444.8522292
Manufacture of pulp and paper,Manufacture of leather products,901.7499592
Manufacture of pulp and paper,Manufacture of footwear,11.70081406
Manufacture of pulp and paper,Sawmilling and planing of wood,4013.125011
Manufacture of pulp and paper,Manufacture of wood products,4676.503063
Manufacture of pulp and paper,Manufacture of pulp and paper,125598.4779
Manufacture of pulp and paper,Manufacture of containers of paper,58443.95331
Manufacture of pulp and paper,Manufacture of other paper products,13907.47212
Manufacture of pulp and paper,Printing,59821.21002
Manufacture of pulp and paper,Manufacture of refined petroleum products,4400.080727
Manufacture of pulp and paper,Manufacture of basic chemicals,9487.534915
Manufacture of pulp and paper,Manufacture of paints,143.8309755
Manufacture of pulp and paper,Manufacture of pharmaceutical products,1370.851618
Manufacture of pulp and paper,Manufacture of soap detergents and toilet products,1056.646035
Manufacture of pulp and paper,Manufacture of other chemical products,1767.839394
Manufacture of pulp and paper,Manufacture of rubber products,356.9324944
Manufacture of pulp and paper,Manufacture of plastic products,4799.082211
Manufacture of pulp and paper,Manufacture of glass and glass products,1383.051013
Manufacture of pulp and paper,Manufacture of cement,3807.164239
Manufacture of pulp and paper,Manufacture of concrete and concrete products,5776.026699
Manufacture of pulp and paper,Manufacture of basic iron and steel,2449.433887
Manufacture of pulp and paper,Manufacture of other basic metals,271.5187847
Manufacture of pulp and paper,Manufacture of fabricated metal products,4554.635028
Manufacture of pulp and paper,Manufacture of industrial and domestic machinery and equipment,2081.697445
Manufacture of pulp and paper,Manufacture of electric and electronic machinery and equipment,1929.777936
Manufacture of pulp and paper,Manufacture of transport equipment,515.4956477
Manufacture of pulp and paper,Manufacture of furniture,1259.084014
Manufacture of pulp and paper,Repair and installation of machinery and other manufacturing,731.2302238
Manufacture of pulp and paper,Electric power generation,25.49816908
Manufacture of pulp and paper,Transmission of electric power,94.25699509
Manufacture of pulp and paper,Distribution of electric power,247079.0906
Manufacture of pulp and paper,Gas and steam manufacture and supply,1565.587873
Manufacture of pulp and paper,Water collection treatment and supply,232.2509056
Manufacture of pulp and paper,Waste collection and recycling activities,81.71796747
Manufacture of pulp and paper,Construction of residential buildings,3593.627256
Manufacture of pulp and paper,Construction of non-residential buildings,1373.656379
Manufacture of pulp and paper,Civil engineering,1835.586352
Manufacture of pulp and paper,Specialized construction activities,2147.575146
Manufacture of pulp and paper,Wholesale and retail trade of motor vehicles,2382.998093
Manufacture of pulp and paper,Wholesale trade,28831.70195
Manufacture of pulp and paper,Retail trade,3212.901962
Manufacture of pulp and paper,Accommodation,421.5729082
Manufacture of pulp and paper,Food and beverage service activities,3823.409064
Manufacture of pulp and paper,Transport via railways,17.96233261
Manufacture of pulp and paper,Other passenger land transport,52.58655694
Manufacture of pulp and paper,Freight transport by road,466.3440249
Manufacture of pulp and paper,Transport via pipeline,10.06590351
Manufacture of pulp and paper,Water transport,41.38377265
Manufacture of pulp and paper,Air transport,119.9529094
Manufacture of pulp and paper,Warehousing,27.26838825
Manufacture of pulp and paper,Support activities for land transport,32.06769654
Manufacture of pulp and paper,Other support activities for transport,785.5408012
Manufacture of pulp and paper,Postal and courier activities,38.56767698
Manufacture of pulp and paper,Wireless telecommunications,102.4587465
Manufacture of pulp and paper,Wired telecommunications,100.3838103
Manufacture of pulp and paper,Other telecommunications activities,101.5597301
Manufacture of pulp and paper,Information services activities,361.8440379
Manufacture of pulp and paper,Publishing activities,19139.35293
Manufacture of pulp and paper,Banking monetary intermediation,1068.758289
Manufacture of pulp and paper,Insurance activities,2279.886338
Manufacture of pulp and paper,Auxiliary financial activities,2183.650167
Manufacture of pulp and paper,Real estate activities,83.17297286
Manufacture of pulp and paper,Housing services,0
Manufacture of pulp and paper,Legal and accounting services,123.4130203
Manufacture of pulp and paper,Architectural and engineering services,8442.043252
Manufacture of pulp and paper,Other professional activities,1008.039378
Manufacture of pulp and paper,Rental and leasing activities,326.0356295
Manufacture of pulp and paper,Business support activities,5638.311142
Manufacture of pulp and paper,Public administration,5158.413255
Manufacture of pulp and paper,Public education,4025.943659
Manufacture of pulp and paper,Private education,2063.683247
Manufacture of pulp and paper,Public health activities,4039.229857
Manufacture of pulp and paper,Private health activities,5871.607075
Manufacture of pulp and paper,Activities of organizations,4106.495871
Manufacture of pulp and paper,Artistic and entertainment activities,867.6816962
Manufacture of pulp and paper,Other personal services,295.6396164
Manufacture of containers of paper,Cultivation of annual crops,124.4446678
Manufacture of containers of paper,Cultivation of vegetables,43.38312954
Manufacture of containers of paper,Cultivation of grapes,134.6682525
Manufacture of containers of paper,Cultivation of other fruits,305.4332883
Manufacture of containers of paper,Cattle bredding,358.1440626
Manufacture of containers of paper,Pigs breeding,168.2862979
Manufacture of containers of paper,Chicken breeding,333.4789943
Manufacture of containers of paper,Breeding of other animals,37.42647742
Manufacture of containers of paper,Farming services,326.9814654
Manufacture of containers of paper,Forestry,24.26508556
Manufacture of containers of paper,Aquaculture,125.9649703
Manufacture of containers of paper,Extractive fishing,31.2164886
Manufacture of containers of paper,Coal mining,2.259623542
Manufacture of containers of paper,Crude oil and gas mining,24.44849928
Manufacture of containers of paper,Copper mining,794.7709833
Manufacture of containers of paper,Iron mining,74.27514654
Manufacture of containers of paper,Mining of other metals,72.24433062
Manufacture of containers of paper,Other mining activities and mining services,53.13734109
Manufacture of containers of paper,Processing and preserving of meat,23783.77755
Manufacture of containers of paper,Processing and preserving of fishmeal and fish oil,590.4921871
Manufacture of containers of paper,Processing and preserving of fish,26114.44098
Manufacture of containers of paper,Processing and preserving of fruits and vegetables,36367.66254
Manufacture of containers of paper,Manufacture of vegetable and animal oils,1847.18828
Manufacture of containers of paper,Manufacture of dairy products,58652.13358
Manufacture of containers of paper,Manufacture of grain mill products,1842.181246
Manufacture of containers of paper,Manufacture of prepared animal feeds,298.2874877
Manufacture of containers of paper,Manufacture of bakery products,17633.53388
Manufacture of containers of paper,Manufacture of macaroni noodles and other related products,1149.819289
Manufacture of containers of paper,Manufacture of other food products,25608.01716
Manufacture of containers of paper,Distilling rectifying and blending of spirits,6483.446534
Manufacture of containers of paper,Manufacture of wines,99952.75569
Manufacture of containers of paper,Manufacture of beers,6818.466244
Manufacture of containers of paper,Manufacture of soft drinks,12415.43188
Manufacture of containers of paper,Manufacture of tobacco products,10350.16162
Manufacture of containers of paper,Manufacture of textile products,3472.802589
Manufacture of containers of paper,Manufacture of wearing apparel,2696.190792
Manufacture of containers of paper,Manufacture of leather products,179.8675576
Manufacture of containers of paper,Manufacture of footwear,1019.794981
Manufacture of containers of paper,Sawmilling and planing of wood,41.90173275
Manufacture of containers of paper,Manufacture of wood products,713.7833317
Manufacture of containers of paper,Manufacture of pulp and paper,1019.583645
Manufacture of containers of paper,Manufacture of containers of paper,11505.01197
Manufacture of containers of paper,Manufacture of other paper products,5083.731609
Manufacture of containers of paper,Printing,15000.70139
Manufacture of containers of paper,Manufacture of refined petroleum products,68.58854688
Manufacture of containers of paper,Manufacture of basic chemicals,1488.817613
Manufacture of containers of paper,Manufacture of paints,66.05000862
Manufacture of containers of paper,Manufacture of pharmaceutical products,34625.21921
Manufacture of containers of paper,Manufacture of soap detergents and toilet products,10787.49075
Manufacture of containers of paper,Manufacture of other chemical products,2960.535829
Manufacture of containers of paper,Manufacture of rubber products,40.37406024
Manufacture of containers of paper,Manufacture of plastic products,3326.856107
Manufacture of containers of paper,Manufacture of glass and glass products,4639.283414
Manufacture of containers of paper,Manufacture of cement,3416.424935
Manufacture of containers of paper,Manufacture of concrete and concrete products,10201.96398
Manufacture of containers of paper,Manufacture of basic iron and steel,1038.831925
Manufacture of containers of paper,Manufacture of other basic metals,129.8613317
Manufacture of containers of paper,Manufacture of fabricated metal products,2551.973185
Manufacture of containers of paper,Manufacture of industrial and domestic machinery and equipment,7238.416405
Manufacture of containers of paper,Manufacture of electric and electronic machinery and equipment,3850.224083
Manufacture of containers of paper,Manufacture of transport equipment,20.92098002
Manufacture of containers of paper,Manufacture of furniture,4878.839577
Manufacture of containers of paper,Repair and installation of machinery and other manufacturing,831.4217589
Manufacture of containers of paper,Electric power generation,242.2488817
Manufacture of containers of paper,Transmission of electric power,41.76972285
Manufacture of containers of paper,Distribution of electric power,89.43268124
Manufacture of containers of paper,Gas and steam manufacture and supply,68.70669446
Manufacture of containers of paper,Water collection treatment and supply,62.52344911
Manufacture of containers of paper,Waste collection and recycling activities,210.4501653
Manufacture of containers of paper,Construction of residential buildings,1038.315112
Manufacture of containers of paper,Construction of non-residential buildings,386.4811349
Manufacture of containers of paper,Civil engineering,1047.283061
Manufacture of containers of paper,Specialized construction activities,1783.336597
Manufacture of containers of paper,Wholesale and retail trade of motor vehicles,2130.765993
Manufacture of containers of paper,Wholesale trade,101364.5135
Manufacture of containers of paper,Retail trade,45530.16988
Manufacture of containers of paper,Accommodation,150.8599739
Manufacture of containers of paper,Food and beverage service activities,988.4179377
Manufacture of containers of paper,Transport via railways,30.21758235
Manufacture of containers of paper,Other passenger land transport,202.52199
Manufacture of containers of paper,Freight transport by road,326.5705289
Manufacture of containers of paper,Transport via pipeline,15.50218562
Manufacture of containers of paper,Water transport,27.09444837
Manufacture of containers of paper,Air transport,54.90046203
Manufacture of containers of paper,Warehousing,7.18524182
Manufacture of containers of paper,Support activities for land transport,8.518785359
Manufacture of containers of paper,Other support activities for transport,234.8747941
Manufacture of containers of paper,Postal and courier activities,271.0184646
Manufacture of containers of paper,Wireless telecommunications,203.8969632
Manufacture of containers of paper,Wired telecommunications,151.5803382
Manufacture of containers of paper,Other telecommunications activities,262.2024685
Manufacture of containers of paper,Information services activities,207.7232833
Manufacture of containers of paper,Publishing activities,964.4787565
Manufacture of containers of paper,Banking monetary intermediation,701.3606192
Manufacture of containers of paper,Insurance activities,615.1414256
Manufacture of containers of paper,Auxiliary financial activities,653.7569558
Manufacture of containers of paper,Real estate activities,265.0362982
Manufacture of containers of paper,Housing services,0
Manufacture of containers of paper,Legal and accounting services,148.294069
Manufacture of containers of paper,Architectural and engineering services,2818.172979
Manufacture of containers of paper,Other professional activities,793.9405127
Manufacture of containers of paper,Rental and leasing activities,419.3050468
Manufacture of containers of paper,Business support activities,1674.984229
Manufacture of containers of paper,Public administration,1237.726006
Manufacture of containers of paper,Public education,926.3546335
Manufacture of containers of paper,Private education,536.001756
Manufacture of containers of paper,Public health activities,1054.299623
Manufacture of containers of paper,Private health activities,1466.316158
Manufacture of containers of paper,Activities of organizations,938.1511604
Manufacture of containers of paper,Artistic and entertainment activities,261.0212863
Manufacture of containers of paper,Other personal services,162.988433
Manufacture of other paper products,Cultivation of annual crops,306.024604
Manufacture of other paper products,Cultivation of vegetables,133.7377441
Manufacture of other paper products,Cultivation of grapes,487.407392
Manufacture of other paper products,Cultivation of other fruits,1146.341361
Manufacture of other paper products,Cattle bredding,1973.828761
Manufacture of other paper products,Pigs breeding,965.8973999
Manufacture of other paper products,Chicken breeding,1924.57249
Manufacture of other paper products,Breeding of other animals,203.264231
Manufacture of other paper products,Farming services,263.7117803
Manufacture of other paper products,Forestry,10.32187824
Manufacture of other paper products,Aquaculture,25.89314528
Manufacture of other paper products,Extractive fishing,9.997442974
Manufacture of other paper products,Coal mining,0.973417596
Manufacture of other paper products,Crude oil and gas mining,86.2040918
Manufacture of other paper products,Copper mining,180.2534946
Manufacture of other paper products,Iron mining,52.50269224
Manufacture of other paper products,Mining of other metals,54.8574248
Manufacture of other paper products,Other mining activities and mining services,4.537155363
Manufacture of other paper products,Processing and preserving of meat,488.343118
Manufacture of other paper products,Processing and preserving of fishmeal and fish oil,100.7951912
Manufacture of other paper products,Processing and preserving of fish,911.414462
Manufacture of other paper products,Processing and preserving of fruits and vegetables,700.5014103
Manufacture of other paper products,Manufacture of vegetable and animal oils,127.9526631
Manufacture of other paper products,Manufacture of dairy products,3950.07727
Manufacture of other paper products,Manufacture of grain mill products,199.3184907
Manufacture of other paper products,Manufacture of prepared animal feeds,598.7276603
Manufacture of other paper products,Manufacture of bakery products,828.1413131
Manufacture of other paper products,Manufacture of macaroni noodles and other related products,207.4846777
Manufacture of other paper products,Manufacture of other food products,1099.349864
Manufacture of other paper products,Distilling rectifying and blending of spirits,955.9408536
Manufacture of other paper products,Manufacture of wines,82.8853923
Manufacture of other paper products,Manufacture of beers,5149.016301
Manufacture of other paper products,Manufacture of soft drinks,4132.471038
Manufacture of other paper products,Manufacture of tobacco products,3562.362622
Manufacture of other paper products,Manufacture of textile products,665.3937239
Manufacture of other paper products,Manufacture of wearing apparel,515.9101484
Manufacture of other paper products,Manufacture of leather products,1072.277243
Manufacture of other paper products,Manufacture of footwear,10.70435774
Manufacture of other paper products,Sawmilling and planing of wood,19.41206089
Manufacture of other paper products,Manufacture of wood products,15.57914736
Manufacture of other paper products,Manufacture of pulp and paper,3305.687553
Manufacture of other paper products,Manufacture of containers of paper,68474.6004
Manufacture of other paper products,Manufacture of other paper products,3019.624579
Manufacture of other paper products,Printing,68460.70343
Manufacture of other paper products,Manufacture of refined petroleum products,23.64328588
Manufacture of other paper products,Manufacture of basic chemicals,339.6936068
Manufacture of other paper products,Manufacture of paints,169.4927915
Manufacture of other paper products,Manufacture of pharmaceutical products,1366.832612
Manufacture of other paper products,Manufacture of soap detergents and toilet products,1181.5443
Manufacture of other paper products,Manufacture of other chemical products,320.2431191
Manufacture of other paper products,Manufacture of rubber products,143.8438637
Manufacture of other paper products,Manufacture of plastic products,80.09653869
Manufacture of other paper products,Manufacture of glass and glass products,45.36849751
Manufacture of other paper products,Manufacture of cement,11.30567507
Manufacture of other paper products,Manufacture of concrete and concrete products,974.0865999
Manufacture of other paper products,Manufacture of basic iron and steel,231.8787605
Manufacture of other paper products,Manufacture of other basic metals,14.93330298
Manufacture of other paper products,Manufacture of fabricated metal products,484.8731227
Manufacture of other paper products,Manufacture of industrial and domestic machinery and equipment,1090.945111
Manufacture of other paper products,Manufacture of electric and electronic machinery and equipment,2248.874857
Manufacture of other paper products,Manufacture of transport equipment,12.28258334
Manufacture of other paper products,Manufacture of furniture,747.9146278
Manufacture of other paper products,Repair and installation of machinery and other manufacturing,827.4762093
Manufacture of other paper products,Electric power generation,35.19130824
Manufacture of other paper products,Transmission of electric power,109.6713967
Manufacture of other paper products,Distribution of electric power,10.37026786
Manufacture of other paper products,Gas and steam manufacture and supply,230.3227707
Manufacture of other paper products,Water collection treatment and supply,255.6788213
Manufacture of other paper products,Waste collection and recycling activities,88.21876281
Manufacture of other paper products,Construction of residential buildings,4082.770427
Manufacture of other paper products,Construction of non-residential buildings,1544.329193
Manufacture of other paper products,Civil engineering,1625.792956
Manufacture of other paper products,Specialized construction activities,2770.3739
Manufacture of other paper products,Wholesale and retail trade of motor vehicles,2873.613758
Manufacture of other paper products,Wholesale trade,2246.414145
Manufacture of other paper products,Retail trade,3757.654855
Manufacture of other paper products,Accommodation,498.5085714
Manufacture of other paper products,Food and beverage service activities,4510.682281
Manufacture of other paper products,Transport via railways,18.03804036
Manufacture of other paper products,Other passenger land transport,53.88853839
Manufacture of other paper products,Freight transport by road,75.61472028
Manufacture of other paper products,Transport via pipeline,11.78960338
Manufacture of other paper products,Water transport,3.970551028
Manufacture of other paper products,Air transport,34.57513381
Manufacture of other paper products,Warehousing,18.68300456
Manufacture of other paper products,Support activities for land transport,4.861372665
Manufacture of other paper products,Other support activities for transport,791.6631225
Manufacture of other paper products,Postal and courier activities,39.59476813
Manufacture of other paper products,Wireless telecommunications,144.7676562
Manufacture of other paper products,Wired telecommunications,90.17408782
Manufacture of other paper products,Other telecommunications activities,112.0561959
Manufacture of other paper products,Information services activities,224.0778748
Manufacture of other paper products,Publishing activities,4794.391644
Manufacture of other paper products,Banking monetary intermediation,1164.514336
Manufacture of other paper products,Insurance activities,2624.73741
Manufacture of other paper products,Auxiliary financial activities,2634.41082
Manufacture of other paper products,Real estate activities,25.80795083
Manufacture of other paper products,Housing services,0
Manufacture of other paper products,Legal and accounting services,27.28532459
Manufacture of other paper products,Architectural and engineering services,9806.421606
Manufacture of other paper products,Other professional activities,1090.152083
Manufacture of other paper products,Rental and leasing activities,239.4345123
Manufacture of other paper products,Business support activities,6510.139893
Manufacture of other paper products,Public administration,5948.954102
Manufacture of other paper products,Public education,4799.170061
Manufacture of other paper products,Private education,2473.207937
Manufacture of other paper products,Public health activities,4822.474055
Manufacture of other paper products,Private health activities,6984.536983
Manufacture of other paper products,Activities of organizations,4921.814157
Manufacture of other paper products,Artistic and entertainment activities,938.02111
Manufacture of other paper products,Other personal services,373.0017878
Printing,Cultivation of annual crops,229.0071075
Printing,Cultivation of vegetables,124.6710999
Printing,Cultivation of grapes,420.3166858
Printing,Cultivation of other fruits,1498.451805
Printing,Cattle bredding,289.5934523
Printing,Pigs breeding,113.8405096
Printing,Chicken breeding,242.539486
Printing,Breeding of other animals,49.71614361
Printing,Farming services,172.3010675
Printing,Forestry,24.50934585
Printing,Aquaculture,82.28260991
Printing,Extractive fishing,23.57459823
Printing,Coal mining,2.838254814
Printing,Crude oil and gas mining,174.7900927
Printing,Copper mining,2904.633114
Printing,Iron mining,82.41013118
Printing,Mining of other metals,98.60300659
Printing,Other mining activities and mining services,313.9982029
Printing,Processing and preserving of meat,2331.347927
Printing,Processing and preserving of fishmeal and fish oil,86.77599108
Printing,Processing and preserving of fish,943.1869512
Printing,Processing and preserving of fruits and vegetables,2525.532656
Printing,Manufacture of vegetable and animal oils,98.19217759
Printing,Manufacture of dairy products,5164.289443
Printing,Manufacture of grain mill products,161.8590971
Printing,Manufacture of prepared animal feeds,358.984859
Printing,Manufacture of bakery products,1268.266303
Printing,Manufacture of macaroni noodles and other related products,168.4577798
Printing,Manufacture of other food products,1671.393303
Printing,Distilling rectifying and blending of spirits,217.9481748
Printing,Manufacture of wines,12770.38933
Printing,Manufacture of beers,311.985579
Printing,Manufacture of soft drinks,619.6477742
Printing,Manufacture of tobacco products,484.7405716
Printing,Manufacture of textile products,645.587067
Printing,Manufacture of wearing apparel,360.6587498
Printing,Manufacture of leather products,31.70541714
Printing,Manufacture of footwear,64.33947548
Printing,Sawmilling and planing of wood,249.8929676
Printing,Manufacture of wood products,149.2145234
Printing,Manufacture of pulp and paper,198.7145189
Printing,Manufacture of containers of paper,2916.933521
Printing,Manufacture of other paper products,670.0082874
Printing,Printing,13419.10201
Printing,Manufacture of refined petroleum products,62.19728203
Printing,Manufacture of basic chemicals,170.2784108
Printing,Manufacture of paints,25.57819207
Printing,Manufacture of pharmaceutical products,2468.408551
Printing,Manufacture of soap detergents and toilet products,1081.102881
Printing,Manufacture of other chemical products,232.9511137
Printing,Manufacture of rubber products,345.8367002
Printing,Manufacture of plastic products,1829.421658
Printing,Manufacture of glass and glass products,197.9063666
Printing,Manufacture of cement,167.4741349
Printing,Manufacture of concrete and concrete products,827.6462122
Printing,Manufacture of basic iron and steel,204.1185447
Printing,Manufacture of other basic metals,69.65132697
Printing,Manufacture of fabricated metal products,1041.71215
Printing,Manufacture of industrial and domestic machinery and equipment,914.3497558
Printing,Manufacture of electric and electronic machinery and equipment,200.4523819
Printing,Manufacture of transport equipment,213.8272959
Printing,Manufacture of furniture,482.3129736
Printing,Repair and installation of machinery and other manufacturing,307.8083916
Printing,Electric power generation,1413.295193
Printing,Transmission of electric power,67.91915887
Printing,Distribution of electric power,408.2795504
Printing,Gas and steam manufacture and supply,82.79464533
Printing,Water collection treatment and supply,143.8229012
Printing,Waste collection and recycling activities,2233.481058
Printing,Construction of residential buildings,6302.695293
Printing,Construction of non-residential buildings,1594.934623
Printing,Civil engineering,3779.297173
Printing,Specialized construction activities,1308.492481
Printing,Wholesale and retail trade of motor vehicles,22242.34737
Printing,Wholesale trade,65938.63459
Printing,Retail trade,56410.06861
Printing,Accommodation,2509.980305
Printing,Food and beverage service activities,8565.827018
Printing,Transport via railways,73.81716554
Printing,Other passenger land transport,5089.58869
Printing,Freight transport by road,3123.447643
Printing,Transport via pipeline,15.91070393
Printing,Water transport,9.420331567
Printing,Air transport,107.298209
Printing,Warehousing,66.16386313
Printing,Support activities for land transport,5.171027992
Printing,Other support activities for transport,136.0692828
Printing,Postal and courier activities,11.70765171
Printing,Wireless telecommunications,232.7124328
Printing,Wired telecommunications,2835.458362
Printing,Other telecommunications activities,22409.52964
Printing,Information services activities,5311.71619
Printing,Publishing activities,36635.01661
Printing,Banking monetary intermediation,24583.37702
Printing,Insurance activities,3346.707382
Printing,Auxiliary financial activities,27388.00104
Printing,Real estate activities,2453.684216
Printing,Housing services,0
Printing,Legal and accounting services,4596.229469
Printing,Architectural and engineering services,19854.9065
Printing,Other professional activities,54141.7707
Printing,Rental and leasing activities,17457.04766
Printing,Business support activities,26035.91224
Printing,Public administration,32394.26196
Printing,Public education,22540.61342
Printing,Private education,24067.80906
Printing,Public health activities,13784.20881
Printing,Private health activities,20652.75684
Printing,Activities of organizations,18447.08563
Printing,Artistic and entertainment activities,7407.681295
Printing,Other personal services,7089.408211
Manufacture of refined petroleum products,Cultivation of annual crops,13683.89971
Manufacture of refined petroleum products,Cultivation of vegetables,10885.59285
Manufacture of refined petroleum products,Cultivation of grapes,15400.56719
Manufacture of refined petroleum products,Cultivation of other fruits,54606.53226
Manufacture of refined petroleum products,Cattle bredding,5103.970827
Manufacture of refined petroleum products,Pigs breeding,2060.976051
Manufacture of refined petroleum products,Chicken breeding,4132.764629
Manufacture of refined petroleum products,Breeding of other animals,444.9797308
Manufacture of refined petroleum products,Farming services,19865.34049
Manufacture of refined petroleum products,Forestry,51603.78547
Manufacture of refined petroleum products,Aquaculture,1519.40482
Manufacture of refined petroleum products,Extractive fishing,23012.86085
Manufacture of refined petroleum products,Coal mining,1690.193184
Manufacture of refined petroleum products,Crude oil and gas mining,1016.208949
Manufacture of refined petroleum products,Copper mining,168956.4936
Manufacture of refined petroleum products,Iron mining,11925.82081
Manufacture of refined petroleum products,Mining of other metals,4328.680397
Manufacture of refined petroleum products,Other mining activities and mining services,12354.61241
Manufacture of refined petroleum products,Processing and preserving of meat,7104.790233
Manufacture of refined petroleum products,Processing and preserving of fishmeal and fish oil,11093.85964
Manufacture of refined petroleum products,Processing and preserving of fish,9475.991756
Manufacture of refined petroleum products,Processing and preserving of fruits and vegetables,11815.12919
Manufacture of refined petroleum products,Manufacture of vegetable and animal oils,298.1003878
Manufacture of refined petroleum products,Manufacture of dairy products,4119.369451
Manufacture of refined petroleum products,Manufacture of grain mill products,497.5456123
Manufacture of refined petroleum products,Manufacture of prepared animal feeds,2959.664248
Manufacture of refined petroleum products,Manufacture of bakery products,8654.147788
Manufacture of refined petroleum products,Manufacture of macaroni noodles and other related products,131.8700959
Manufacture of refined petroleum products,Manufacture of other food products,2155.387245
Manufacture of refined petroleum products,Distilling rectifying and blending of spirits,141.0083236
Manufacture of refined petroleum products,Manufacture of wines,2858.449003
Manufacture of refined petroleum products,Manufacture of beers,2764.954665
Manufacture of refined petroleum products,Manufacture of soft drinks,1649.535884
Manufacture of refined petroleum products,Manufacture of tobacco products,1030.930736
Manufacture of refined petroleum products,Manufacture of textile products,934.1546352
Manufacture of refined petroleum products,Manufacture of wearing apparel,1478.498174
Manufacture of refined petroleum products,Manufacture of leather products,150.5364819
Manufacture of refined petroleum products,Manufacture of footwear,92.99320841
Manufacture of refined petroleum products,Sawmilling and planing of wood,1212.840769
Manufacture of refined petroleum products,Manufacture of wood products,5023.346966
Manufacture of refined petroleum products,Manufacture of pulp and paper,29432.75746
Manufacture of refined petroleum products,Manufacture of containers of paper,1592.808981
Manufacture of refined petroleum products,Manufacture of other paper products,378.8414446
Manufacture of refined petroleum products,Printing,515.7982175
Manufacture of refined petroleum products,Manufacture of refined petroleum products,53514.95655
Manufacture of refined petroleum products,Manufacture of basic chemicals,37886.09142
Manufacture of refined petroleum products,Manufacture of paints,823.9763815
Manufacture of refined petroleum products,Manufacture of pharmaceutical products,921.7490498
Manufacture of refined petroleum products,Manufacture of soap detergents and toilet products,1015.298324
Manufacture of refined petroleum products,Manufacture of other chemical products,2445.617947
Manufacture of refined petroleum products,Manufacture of rubber products,2139.5922
Manufacture of refined petroleum products,Manufacture of plastic products,3432.563337
Manufacture of refined petroleum products,Manufacture of glass and glass products,11014.42525
Manufacture of refined petroleum products,Manufacture of cement,9268.572172
Manufacture of refined petroleum products,Manufacture of concrete and concrete products,16919.95129
Manufacture of refined petroleum products,Manufacture of basic iron and steel,10693.76031
Manufacture of refined petroleum products,Manufacture of other basic metals,491.7927074
Manufacture of refined petroleum products,Manufacture of fabricated metal products,4380.105166
Manufacture of refined petroleum products,Manufacture of industrial and domestic machinery and equipment,2914.685217
Manufacture of refined petroleum products,Manufacture of electric and electronic machinery and equipment,828.5002013
Manufacture of refined petroleum products,Manufacture of transport equipment,654.3026989
Manufacture of refined petroleum products,Manufacture of furniture,1736.31305
Manufacture of refined petroleum products,Repair and installation of machinery and other manufacturing,7380.43158
Manufacture of refined petroleum products,Electric power generation,35282.08755
Manufacture of refined petroleum products,Transmission of electric power,1.136396015
Manufacture of refined petroleum products,Distribution of electric power,626.5849085
Manufacture of refined petroleum products,Gas and steam manufacture and supply,1015.511168
Manufacture of refined petroleum products,Water collection treatment and supply,1096.7596
Manufacture of refined petroleum products,Waste collection and recycling activities,16185.5095
Manufacture of refined petroleum products,Construction of residential buildings,6772.625259
Manufacture of refined petroleum products,Construction of non-residential buildings,3392.1417
Manufacture of refined petroleum products,Civil engineering,56260.27811
Manufacture of refined petroleum products,Specialized construction activities,38467.69895
Manufacture of refined petroleum products,Wholesale and retail trade of motor vehicles,30725.74146
Manufacture of refined petroleum products,Wholesale trade,36326.74228
Manufacture of refined petroleum products,Retail trade,20116.1082
Manufacture of refined petroleum products,Accommodation,6033.117493
Manufacture of refined petroleum products,Food and beverage service activities,6055.837644
Manufacture of refined petroleum products,Transport via railways,7305.759931
Manufacture of refined petroleum products,Other passenger land transport,389896.8279
Manufacture of refined petroleum products,Freight transport by road,331846.0364
Manufacture of refined petroleum products,Transport via pipeline,65.94444331
Manufacture of refined petroleum products,Water transport,4024.294176
Manufacture of refined petroleum products,Air transport,132838.2358
Manufacture of refined petroleum products,Warehousing,1055.146041
Manufacture of refined petroleum products,Support activities for land transport,857.1881236
Manufacture of refined petroleum products,Other support activities for transport,6227.084184
Manufacture of refined petroleum products,Postal and courier activities,32.15072725
Manufacture of refined petroleum products,Wireless telecommunications,2025.949755
Manufacture of refined petroleum products,Wired telecommunications,504.7378622
Manufacture of refined petroleum products,Other telecommunications activities,2615.897806
Manufacture of refined petroleum products,Information services activities,673.6585436
Manufacture of refined petroleum products,Publishing activities,596.4051741
Manufacture of refined petroleum products,Banking monetary intermediation,318.1119542
Manufacture of refined petroleum products,Insurance activities,90.77519744
Manufacture of refined petroleum products,Auxiliary financial activities,199.4148057
Manufacture of refined petroleum products,Real estate activities,2382.880134
Manufacture of refined petroleum products,Housing services,0
Manufacture of refined petroleum products,Legal and accounting services,558.5145099
Manufacture of refined petroleum products,Architectural and engineering services,13706.98213
Manufacture of refined petroleum products,Other professional activities,5667.837876
Manufacture of refined petroleum products,Rental and leasing activities,22771.04335
Manufacture of refined petroleum products,Business support activities,15777.30533
Manufacture of refined petroleum products,Public administration,24174.04314
Manufacture of refined petroleum products,Public education,2193.734138
Manufacture of refined petroleum products,Private education,2697.426321
Manufacture of refined petroleum products,Public health activities,12827.66977
Manufacture of refined petroleum products,Private health activities,19874.07085
Manufacture of refined petroleum products,Activities of organizations,10732.69145
Manufacture of refined petroleum products,Artistic and entertainment activities,2043.250755
Manufacture of refined petroleum products,Other personal services,8612.159187
Manufacture of basic chemicals,Cultivation of annual crops,24145.29183
Manufacture of basic chemicals,Cultivation of vegetables,14997.95814
Manufacture of basic chemicals,Cultivation of grapes,31785.69751
Manufacture of basic chemicals,Cultivation of other fruits,28627.60959
Manufacture of basic chemicals,Cattle bredding,3481.398888
Manufacture of basic chemicals,Pigs breeding,132.1993409
Manufacture of basic chemicals,Chicken breeding,264.4441311
Manufacture of basic chemicals,Breeding of other animals,351.5732378
Manufacture of basic chemicals,Farming services,1659.331267
Manufacture of basic chemicals,Forestry,2975.056114
Manufacture of basic chemicals,Aquaculture,16018.34953
Manufacture of basic chemicals,Extractive fishing,441.7114023
Manufacture of basic chemicals,Coal mining,198.1178443
Manufacture of basic chemicals,Crude oil and gas mining,667.1573982
Manufacture of basic chemicals,Copper mining,259384.4557
Manufacture of basic chemicals,Iron mining,335.0627771
Manufacture of basic chemicals,Mining of other metals,8979.171523
Manufacture of basic chemicals,Other mining activities and mining services,4802.243438
Manufacture of basic chemicals,Processing and preserving of meat,12342.3274
Manufacture of basic chemicals,Processing and preserving of fishmeal and fish oil,366.6320191
Manufacture of basic chemicals,Processing and preserving of fish,1542.499549
Manufacture of basic chemicals,Processing and preserving of fruits and vegetables,9384.658604
Manufacture of basic chemicals,Manufacture of vegetable and animal oils,381.1269305
Manufacture of basic chemicals,Manufacture of dairy products,1394.4575
Manufacture of basic chemicals,Manufacture of grain mill products,740.6144491
Manufacture of basic chemicals,Manufacture of prepared animal feeds,12472.99163
Manufacture of basic chemicals,Manufacture of bakery products,3274.43519
Manufacture of basic chemicals,Manufacture of macaroni noodles and other related products,47.54377616
Manufacture of basic chemicals,Manufacture of other food products,11120.89352
Manufacture of basic chemicals,Distilling rectifying and blending of spirits,261.7914989
Manufacture of basic chemicals,Manufacture of wines,2532.540923
Manufacture of basic chemicals,Manufacture of beers,1024.179018
Manufacture of basic chemicals,Manufacture of soft drinks,4450.638684
Manufacture of basic chemicals,Manufacture of tobacco products,233.861355
Manufacture of basic chemicals,Manufacture of textile products,10867.38665
Manufacture of basic chemicals,Manufacture of wearing apparel,1284.746311
Manufacture of basic chemicals,Manufacture of leather products,4431.663351
Manufacture of basic chemicals,Manufacture of footwear,2895.444117
Manufacture of basic chemicals,Sawmilling and planing of wood,1326.35347
Manufacture of basic chemicals,Manufacture of wood products,32867.96037
Manufacture of basic chemicals,Manufacture of pulp and paper,60542.48441
Manufacture of basic chemicals,Manufacture of containers of paper,18375.94246
Manufacture of basic chemicals,Manufacture of other paper products,3863.244004
Manufacture of basic chemicals,Printing,1064.098587
Manufacture of basic chemicals,Manufacture of refined petroleum products,1183.933358
Manufacture of basic chemicals,Manufacture of basic chemicals,95543.34254
Manufacture of basic chemicals,Manufacture of paints,4711.905046
Manufacture of basic chemicals,Manufacture of pharmaceutical products,9906.307035
Manufacture of basic chemicals,Manufacture of soap detergents and toilet products,10203.88826
Manufacture of basic chemicals,Manufacture of other chemical products,69624.57593
Manufacture of basic chemicals,Manufacture of rubber products,13921.6289
Manufacture of basic chemicals,Manufacture of plastic products,83124.67811
Manufacture of basic chemicals,Manufacture of glass and glass products,6644.766032
Manufacture of basic chemicals,Manufacture of cement,8310.753883
Manufacture of basic chemicals,Manufacture of concrete and concrete products,20049.83011
Manufacture of basic chemicals,Manufacture of basic iron and steel,6671.49843
Manufacture of basic chemicals,Manufacture of other basic metals,2990.127803
Manufacture of basic chemicals,Manufacture of fabricated metal products,18960.89357
Manufacture of basic chemicals,Manufacture of industrial and domestic machinery and equipment,12056.24208
Manufacture of basic chemicals,Manufacture of electric and electronic machinery and equipment,4719.909536
Manufacture of basic chemicals,Manufacture of transport equipment,1658.901855
Manufacture of basic chemicals,Manufacture of furniture,2551.435155
Manufacture of basic chemicals,Repair and installation of machinery and other manufacturing,4794.861623
Manufacture of basic chemicals,Electric power generation,1253.569407
Manufacture of basic chemicals,Transmission of electric power,8.773942729
Manufacture of basic chemicals,Distribution of electric power,97.26470595
Manufacture of basic chemicals,Gas and steam manufacture and supply,1633.260289
Manufacture of basic chemicals,Water collection treatment and supply,8396.579383
Manufacture of basic chemicals,Waste collection and recycling activities,1689.58938
Manufacture of basic chemicals,Construction of residential buildings,7771.034919
Manufacture of basic chemicals,Construction of non-residential buildings,6260.577219
Manufacture of basic chemicals,Civil engineering,25897.29491
Manufacture of basic chemicals,Specialized construction activities,13499.53575
Manufacture of basic chemicals,Wholesale and retail trade of motor vehicles,2688.019689
Manufacture of basic chemicals,Wholesale trade,16788.81346
Manufacture of basic chemicals,Retail trade,22124.31645
Manufacture of basic chemicals,Accommodation,533.0383014
Manufacture of basic chemicals,Food and beverage service activities,2141.605172
Manufacture of basic chemicals,Transport via railways,161.7008293
Manufacture of basic chemicals,Other passenger land transport,1013.212702
Manufacture of basic chemicals,Freight transport by road,2422.722994
Manufacture of basic chemicals,Transport via pipeline,0.558638457
Manufacture of basic chemicals,Water transport,324.4663874
Manufacture of basic chemicals,Air transport,881.0802841
Manufacture of basic chemicals,Warehousing,256.3915843
Manufacture of basic chemicals,Support activities for land transport,424.3317694
Manufacture of basic chemicals,Other support activities for transport,792.1519646
Manufacture of basic chemicals,Postal and courier activities,53.08979093
Manufacture of basic chemicals,Wireless telecommunications,3128.053719
Manufacture of basic chemicals,Wired telecommunications,868.7883757
Manufacture of basic chemicals,Other telecommunications activities,2170.946302
Manufacture of basic chemicals,Information services activities,2377.733832
Manufacture of basic chemicals,Publishing activities,1292.893584
Manufacture of basic chemicals,Banking monetary intermediation,3919.470887
Manufacture of basic chemicals,Insurance activities,2319.307153
Manufacture of basic chemicals,Auxiliary financial activities,7180.232739
Manufacture of basic chemicals,Real estate activities,2300.464505
Manufacture of basic chemicals,Housing services,0
Manufacture of basic chemicals,Legal and accounting services,1643.468554
Manufacture of basic chemicals,Architectural and engineering services,3829.705122
Manufacture of basic chemicals,Other professional activities,6958.233556
Manufacture of basic chemicals,Rental and leasing activities,1650.037609
Manufacture of basic chemicals,Business support activities,6686.036373
Manufacture of basic chemicals,Public administration,6351.209024
Manufacture of basic chemicals,Public education,1763.173559
Manufacture of basic chemicals,Private education,4438.503718
Manufacture of basic chemicals,Public health activities,4800.125087
Manufacture of basic chemicals,Private health activities,14348.666
Manufacture of basic chemicals,Activities of organizations,2172.658413
Manufacture of basic chemicals,Artistic and entertainment activities,992.1912364
Manufacture of basic chemicals,Other personal services,439.8945292
Manufacture of paints,Cultivation of annual crops,1341.485736
Manufacture of paints,Cultivation of vegetables,176.4824727
Manufacture of paints,Cultivation of grapes,370.7174393
Manufacture of paints,Cultivation of other fruits,401.356543
Manufacture of paints,Cattle bredding,100.234734
Manufacture of paints,Pigs breeding,12.19449234
Manufacture of paints,Chicken breeding,15.90699638
Manufacture of paints,Breeding of other animals,9.990360365
Manufacture of paints,Farming services,229.6981497
Manufacture of paints,Forestry,94.41936798
Manufacture of paints,Aquaculture,221.3984099
Manufacture of paints,Extractive fishing,21.90927796
Manufacture of paints,Coal mining,4.483837471
Manufacture of paints,Crude oil and gas mining,10.73571092
Manufacture of paints,Copper mining,6918.927323
Manufacture of paints,Iron mining,16.84375511
Manufacture of paints,Mining of other metals,110.6666034
Manufacture of paints,Other mining activities and mining services,61.75058902
Manufacture of paints,Processing and preserving of meat,223.3607374
Manufacture of paints,Processing and preserving of fishmeal and fish oil,10.46013383
Manufacture of paints,Processing and preserving of fish,59.36297101
Manufacture of paints,Processing and preserving of fruits and vegetables,137.1637426
Manufacture of paints,Manufacture of vegetable and animal oils,20.78448712
Manufacture of paints,Manufacture of dairy products,89.90710028
Manufacture of paints,Manufacture of grain mill products,71.39038407
Manufacture of paints,Manufacture of prepared animal feeds,216.6543338
Manufacture of paints,Manufacture of bakery products,98.13280154
Manufacture of paints,Manufacture of macaroni noodles and other related products,4.958563926
Manufacture of paints,Manufacture of other food products,302.3386493
Manufacture of paints,Distilling rectifying and blending of spirits,99.39383477
Manufacture of paints,Manufacture of wines,100.2557747
Manufacture of paints,Manufacture of beers,41.33780994
Manufacture of paints,Manufacture of soft drinks,1125.699501
Manufacture of paints,Manufacture of tobacco products,10.99576626
Manufacture of paints,Manufacture of textile products,3596.278346
Manufacture of paints,Manufacture of wearing apparel,145.1409711
Manufacture of paints,Manufacture of leather products,46.56488494
Manufacture of paints,Manufacture of footwear,1196.253508
Manufacture of paints,Sawmilling and planing of wood,47.43263983
Manufacture of paints,Manufacture of wood products,368.3009315
Manufacture of paints,Manufacture of pulp and paper,911.9506103
Manufacture of paints,Manufacture of containers of paper,5370.756876
Manufacture of paints,Manufacture of other paper products,3225.811025
Manufacture of paints,Printing,12965.18418
Manufacture of paints,Manufacture of refined petroleum products,40.52624493
Manufacture of paints,Manufacture of basic chemicals,1380.40586
Manufacture of paints,Manufacture of paints,21052.79421
Manufacture of paints,Manufacture of pharmaceutical products,185.2115241
Manufacture of paints,Manufacture of soap detergents and toilet products,7129.322171
Manufacture of paints,Manufacture of other chemical products,734.2532349
Manufacture of paints,Manufacture of rubber products,223.1760446
Manufacture of paints,Manufacture of plastic products,13623.89312
Manufacture of paints,Manufacture of glass and glass products,245.623504
Manufacture of paints,Manufacture of cement,51.79830268
Manufacture of paints,Manufacture of concrete and concrete products,1690.577304
Manufacture of paints,Manufacture of basic iron and steel,446.8670051
Manufacture of paints,Manufacture of other basic metals,205.820985
Manufacture of paints,Manufacture of fabricated metal products,2811.438312
Manufacture of paints,Manufacture of industrial and domestic machinery and equipment,1018.783439
Manufacture of paints,Manufacture of electric and electronic machinery and equipment,2296.683074
Manufacture of paints,Manufacture of transport equipment,1767.130208
Manufacture of paints,Manufacture of furniture,6824.667728
Manufacture of paints,Repair and installation of machinery and other manufacturing,10087.44765
Manufacture of paints,Electric power generation,61.65137987
Manufacture of paints,Transmission of electric power,0.812070423
Manufacture of paints,Distribution of electric power,12.0672098
Manufacture of paints,Gas and steam manufacture and supply,62.92024755
Manufacture of paints,Water collection treatment and supply,94.17636972
Manufacture of paints,Waste collection and recycling activities,27.72290241
Manufacture of paints,Construction of residential buildings,3834.706602
Manufacture of paints,Construction of non-residential buildings,11445.691
Manufacture of paints,Civil engineering,12164.30137
Manufacture of paints,Specialized construction activities,74219.59288
Manufacture of paints,Wholesale and retail trade of motor vehicles,220.8706042
Manufacture of paints,Wholesale trade,902.6766096
Manufacture of paints,Retail trade,1393.651967
Manufacture of paints,Accommodation,104.2113739
Manufacture of paints,Food and beverage service activities,1118.588957
Manufacture of paints,Transport via railways,6.172870989
Manufacture of paints,Other passenger land transport,103.3054455
Manufacture of paints,Freight transport by road,259.1183201
Manufacture of paints,Transport via pipeline,0.101218292
Manufacture of paints,Water transport,20.2157901
Manufacture of paints,Air transport,80.48878101
Manufacture of paints,Warehousing,12.32494088
Manufacture of paints,Support activities for land transport,20.52930124
Manufacture of paints,Other support activities for transport,71.81080018
Manufacture of paints,Postal and courier activities,2.661143819
Manufacture of paints,Wireless telecommunications,304.0036745
Manufacture of paints,Wired telecommunications,98.20494415
Manufacture of paints,Other telecommunications activities,153.6119006
Manufacture of paints,Information services activities,1016.649198
Manufacture of paints,Publishing activities,78.07959734
Manufacture of paints,Banking monetary intermediation,532.6109966
Manufacture of paints,Insurance activities,4408.399812
Manufacture of paints,Auxiliary financial activities,3583.615658
Manufacture of paints,Real estate activities,105.7353555
Manufacture of paints,Housing services,0
Manufacture of paints,Legal and accounting services,104.4447575
Manufacture of paints,Architectural and engineering services,234.6850135
Manufacture of paints,Other professional activities,450.9535051
Manufacture of paints,Rental and leasing activities,151.2283264
Manufacture of paints,Business support activities,389.8286461
Manufacture of paints,Public administration,9103.283653
Manufacture of paints,Public education,1883.314225
Manufacture of paints,Private education,1607.122841
Manufacture of paints,Public health activities,5576.043243
Manufacture of paints,Private health activities,22507.67174
Manufacture of paints,Activities of organizations,7858.492363
Manufacture of paints,Artistic and entertainment activities,94.55548807
Manufacture of paints,Other personal services,44.11376115
Manufacture of pharmaceutical products,Cultivation of annual crops,1003.203018
Manufacture of pharmaceutical products,Cultivation of vegetables,358.9854491
Manufacture of pharmaceutical products,Cultivation of grapes,765.8009867
Manufacture of pharmaceutical products,Cultivation of other fruits,1266.563014
Manufacture of pharmaceutical products,Cattle bredding,42665.81669
Manufacture of pharmaceutical products,Pigs breeding,2886.181572
Manufacture of pharmaceutical products,Chicken breeding,2666.159069
Manufacture of pharmaceutical products,Breeding of other animals,5686.303705
Manufacture of pharmaceutical products,Farming services,424.250741
Manufacture of pharmaceutical products,Forestry,450.6454405
Manufacture of pharmaceutical products,Aquaculture,38465.71023
Manufacture of pharmaceutical products,Extractive fishing,127.6755438
Manufacture of pharmaceutical products,Coal mining,22.73416272
Manufacture of pharmaceutical products,Crude oil and gas mining,32.48898724
Manufacture of pharmaceutical products,Copper mining,6083.888649
Manufacture of pharmaceutical products,Iron mining,164.3304628
Manufacture of pharmaceutical products,Mining of other metals,257.9951713
Manufacture of pharmaceutical products,Other mining activities and mining services,148.2458199
Manufacture of pharmaceutical products,Processing and preserving of meat,2561.083075
Manufacture of pharmaceutical products,Processing and preserving of fishmeal and fish oil,56.31017643
Manufacture of pharmaceutical products,Processing and preserving of fish,1306.208288
Manufacture of pharmaceutical products,Processing and preserving of fruits and vegetables,461.7293903
Manufacture of pharmaceutical products,Manufacture of vegetable and animal oils,134.4315598
Manufacture of pharmaceutical products,Manufacture of dairy products,2708.54314
Manufacture of pharmaceutical products,Manufacture of grain mill products,213.088898
Manufacture of pharmaceutical products,Manufacture of prepared animal feeds,35520.63267
Manufacture of pharmaceutical products,Manufacture of bakery products,583.87822
Manufacture of pharmaceutical products,Manufacture of macaroni noodles and other related products,928.6150895
Manufacture of pharmaceutical products,Manufacture of other food products,5974.809061
Manufacture of pharmaceutical products,Distilling rectifying and blending of spirits,80.64070341
Manufacture of pharmaceutical products,Manufacture of wines,715.3716538
Manufacture of pharmaceutical products,Manufacture of beers,277.0860344
Manufacture of pharmaceutical products,Manufacture of soft drinks,636.5709815
Manufacture of pharmaceutical products,Manufacture of tobacco products,66.71347752
Manufacture of pharmaceutical products,Manufacture of textile products,332.4349971
Manufacture of pharmaceutical products,Manufacture of wearing apparel,571.8420745
Manufacture of pharmaceutical products,Manufacture of leather products,67.76428499
Manufacture of pharmaceutical products,Manufacture of footwear,87.99880168
Manufacture of pharmaceutical products,Sawmilling and planing of wood,295.0514986
Manufacture of pharmaceutical products,Manufacture of wood products,714.9085916
Manufacture of pharmaceutical products,Manufacture of pulp and paper,1237.619694
Manufacture of pharmaceutical products,Manufacture of containers of paper,421.5871886
Manufacture of pharmaceutical products,Manufacture of other paper products,627.9750055
Manufacture of pharmaceutical products,Printing,237.8706651
Manufacture of pharmaceutical products,Manufacture of refined petroleum products,235.4438468
Manufacture of pharmaceutical products,Manufacture of basic chemicals,1343.218241
Manufacture of pharmaceutical products,Manufacture of paints,218.9621717
Manufacture of pharmaceutical products,Manufacture of pharmaceutical products,93392.91896
Manufacture of pharmaceutical products,Manufacture of soap detergents and toilet products,1211.493452
Manufacture of pharmaceutical products,Manufacture of other chemical products,2398.020616
Manufacture of pharmaceutical products,Manufacture of rubber products,236.9683569
Manufacture of pharmaceutical products,Manufacture of plastic products,1865.900643
Manufacture of pharmaceutical products,Manufacture of glass and glass products,253.0810385
Manufacture of pharmaceutical products,Manufacture of cement,258.4178584
Manufacture of pharmaceutical products,Manufacture of concrete and concrete products,808.4208605
Manufacture of pharmaceutical products,Manufacture of basic iron and steel,646.2382206
Manufacture of pharmaceutical products,Manufacture of other basic metals,297.7255393
Manufacture of pharmaceutical products,Manufacture of fabricated metal products,1175.07422
Manufacture of pharmaceutical products,Manufacture of industrial and domestic machinery and equipment,1398.664971
Manufacture of pharmaceutical products,Manufacture of electric and electronic machinery and equipment,300.7534783
Manufacture of pharmaceutical products,Manufacture of transport equipment,259.608507
Manufacture of pharmaceutical products,Manufacture of furniture,548.1598183
Manufacture of pharmaceutical products,Repair and installation of machinery and other manufacturing,368.0803885
Manufacture of pharmaceutical products,Electric power generation,411.8747607
Manufacture of pharmaceutical products,Transmission of electric power,4.131518853
Manufacture of pharmaceutical products,Distribution of electric power,54.83541935
Manufacture of pharmaceutical products,Gas and steam manufacture and supply,444.6365751
Manufacture of pharmaceutical products,Water collection treatment and supply,207.9311528
Manufacture of pharmaceutical products,Waste collection and recycling activities,154.7574207
Manufacture of pharmaceutical products,Construction of residential buildings,1984.818322
Manufacture of pharmaceutical products,Construction of non-residential buildings,1189.184952
Manufacture of pharmaceutical products,Civil engineering,3235.164462
Manufacture of pharmaceutical products,Specialized construction activities,2529.025012
Manufacture of pharmaceutical products,Wholesale and retail trade of motor vehicles,1468.932728
Manufacture of pharmaceutical products,Wholesale trade,5668.752977
Manufacture of pharmaceutical products,Retail trade,7592.084432
Manufacture of pharmaceutical products,Accommodation,394.0704932
Manufacture of pharmaceutical products,Food and beverage service activities,1491.130746
Manufacture of pharmaceutical products,Transport via railways,40.2653899
Manufacture of pharmaceutical products,Other passenger land transport,719.2669442
Manufacture of pharmaceutical products,Freight transport by road,1408.230819
Manufacture of pharmaceutical products,Transport via pipeline,0.373207701
Manufacture of pharmaceutical products,Water transport,122.8767479
Manufacture of pharmaceutical products,Air transport,428.4245659
Manufacture of pharmaceutical products,Warehousing,71.7747785
Manufacture of pharmaceutical products,Support activities for land transport,132.1308283
Manufacture of pharmaceutical products,Other support activities for transport,411.0956928
Manufacture of pharmaceutical products,Postal and courier activities,17.50084285
Manufacture of pharmaceutical products,Wireless telecommunications,1992.093758
Manufacture of pharmaceutical products,Wired telecommunications,561.1607815
Manufacture of pharmaceutical products,Other telecommunications activities,1013.79139
Manufacture of pharmaceutical products,Information services activities,943.3810448
Manufacture of pharmaceutical products,Publishing activities,520.6690903
Manufacture of pharmaceutical products,Banking monetary intermediation,1261.887481
Manufacture of pharmaceutical products,Insurance activities,679.150153
Manufacture of pharmaceutical products,Auxiliary financial activities,438.5356444
Manufacture of pharmaceutical products,Real estate activities,780.1132287
Manufacture of pharmaceutical products,Housing services,0
Manufacture of pharmaceutical products,Legal and accounting services,538.0774909
Manufacture of pharmaceutical products,Architectural and engineering services,1248.476102
Manufacture of pharmaceutical products,Other professional activities,9699.874584
Manufacture of pharmaceutical products,Rental and leasing activities,821.2332936
Manufacture of pharmaceutical products,Business support activities,3236.615728
Manufacture of pharmaceutical products,Public administration,1102.624606
Manufacture of pharmaceutical products,Public education,2176.500552
Manufacture of pharmaceutical products,Private education,1936.658434
Manufacture of pharmaceutical products,Public health activities,166480.1799
Manufacture of pharmaceutical products,Private health activities,219451.0931
Manufacture of pharmaceutical products,Activities of organizations,239.579398
Manufacture of pharmaceutical products,Artistic and entertainment activities,528.4503132
Manufacture of pharmaceutical products,Other personal services,363.0116453
Manufacture of soap detergents and toilet products,Cultivation of annual crops,664.6803882
Manufacture of soap detergents and toilet products,Cultivation of vegetables,246.2796648
Manufacture of soap detergents and toilet products,Cultivation of grapes,526.1294512
Manufacture of soap detergents and toilet products,Cultivation of other fruits,802.8435204
Manufacture of soap detergents and toilet products,Cattle bredding,307.2373115
Manufacture of soap detergents and toilet products,Pigs breeding,52.28520254
Manufacture of soap detergents and toilet products,Chicken breeding,71.19225792
Manufacture of soap detergents and toilet products,Breeding of other animals,30.98537253
Manufacture of soap detergents and toilet products,Farming services,159.1580377
Manufacture of soap detergents and toilet products,Forestry,136.9640685
Manufacture of soap detergents and toilet products,Aquaculture,344.9731826
Manufacture of soap detergents and toilet products,Extractive fishing,77.69913464
Manufacture of soap detergents and toilet products,Coal mining,64.8427253
Manufacture of soap detergents and toilet products,Crude oil and gas mining,15.35498723
Manufacture of soap detergents and toilet products,Copper mining,6417.625165
Manufacture of soap detergents and toilet products,Iron mining,3207.66162
Manufacture of soap detergents and toilet products,Mining of other metals,789.1952254
Manufacture of soap detergents and toilet products,Other mining activities and mining services,90.3667075
Manufacture of soap detergents and toilet products,Processing and preserving of meat,446.8271238
Manufacture of soap detergents and toilet products,Processing and preserving of fishmeal and fish oil,27.28133471
Manufacture of soap detergents and toilet products,Processing and preserving of fish,106.6204166
Manufacture of soap detergents and toilet products,Processing and preserving of fruits and vegetables,284.7401071
Manufacture of soap detergents and toilet products,Manufacture of vegetable and animal oils,104.8248544
Manufacture of soap detergents and toilet products,Manufacture of dairy products,464.469298
Manufacture of soap detergents and toilet products,Manufacture of grain mill products,94.08470907
Manufacture of soap detergents and toilet products,Manufacture of prepared animal feeds,716.7924085
Manufacture of soap detergents and toilet products,Manufacture of bakery products,227.7507912
Manufacture of soap detergents and toilet products,Manufacture of macaroni noodles and other related products,28.24152932
Manufacture of soap detergents and toilet products,Manufacture of other food products,469.6785786
Manufacture of soap detergents and toilet products,Distilling rectifying and blending of spirits,25.22677352
Manufacture of soap detergents and toilet products,Manufacture of wines,371.5010122
Manufacture of soap detergents and toilet products,Manufacture of beers,168.0829425
Manufacture of soap detergents and toilet products,Manufacture of soft drinks,440.4609548
Manufacture of soap detergents and toilet products,Manufacture of tobacco products,67.55082069
Manufacture of soap detergents and toilet products,Manufacture of textile products,360.0738039
Manufacture of soap detergents and toilet products,Manufacture of wearing apparel,355.641867
Manufacture of soap detergents and toilet products,Manufacture of leather products,91.12075793
Manufacture of soap detergents and toilet products,Manufacture of footwear,64.41813322
Manufacture of soap detergents and toilet products,Sawmilling and planing of wood,98.59109089
Manufacture of soap detergents and toilet products,Manufacture of wood products,642.6827327
Manufacture of soap detergents and toilet products,Manufacture of pulp and paper,903.9766587
Manufacture of soap detergents and toilet products,Manufacture of containers of paper,1430.626004
Manufacture of soap detergents and toilet products,Manufacture of other paper products,7796.309724
Manufacture of soap detergents and toilet products,Printing,872.4756123
Manufacture of soap detergents and toilet products,Manufacture of refined petroleum products,202.5571241
Manufacture of soap detergents and toilet products,Manufacture of basic chemicals,900.4761609
Manufacture of soap detergents and toilet products,Manufacture of paints,155.837182
Manufacture of soap detergents and toilet products,Manufacture of pharmaceutical products,256.2320484
Manufacture of soap detergents and toilet products,Manufacture of soap detergents and toilet products,759.5071879
Manufacture of soap detergents and toilet products,Manufacture of other chemical products,917.0987814
Manufacture of soap detergents and toilet products,Manufacture of rubber products,173.9978213
Manufacture of soap detergents and toilet products,Manufacture of plastic products,1406.338408
Manufacture of soap detergents and toilet products,Manufacture of glass and glass products,228.688814
Manufacture of soap detergents and toilet products,Manufacture of cement,94.41533008
Manufacture of soap detergents and toilet products,Manufacture of concrete and concrete products,845.5193969
Manufacture of soap detergents and toilet products,Manufacture of basic iron and steel,341.6653593
Manufacture of soap detergents and toilet products,Manufacture of other basic metals,156.3511226
Manufacture of soap detergents and toilet products,Manufacture of fabricated metal products,494.669439
Manufacture of soap detergents and toilet products,Manufacture of industrial and domestic machinery and equipment,589.2276022
Manufacture of soap detergents and toilet products,Manufacture of electric and electronic machinery and equipment,203.4390556
Manufacture of soap detergents and toilet products,Manufacture of transport equipment,135.1846872
Manufacture of soap detergents and toilet products,Manufacture of furniture,515.3701877
Manufacture of soap detergents and toilet products,Repair and installation of machinery and other manufacturing,714.2050004
Manufacture of soap detergents and toilet products,Electric power generation,181.4681793
Manufacture of soap detergents and toilet products,Transmission of electric power,1.869524141
Manufacture of soap detergents and toilet products,Distribution of electric power,6.190487981
Manufacture of soap detergents and toilet products,Gas and steam manufacture and supply,15.06782533
Manufacture of soap detergents and toilet products,Water collection treatment and supply,2428.473427
Manufacture of soap detergents and toilet products,Waste collection and recycling activities,3265.453938
Manufacture of soap detergents and toilet products,Construction of residential buildings,1272.035965
Manufacture of soap detergents and toilet products,Construction of non-residential buildings,848.7804763
Manufacture of soap detergents and toilet products,Civil engineering,2748.161229
Manufacture of soap detergents and toilet products,Specialized construction activities,2306.268042
Manufacture of soap detergents and toilet products,Wholesale and retail trade of motor vehicles,1605.895459
Manufacture of soap detergents and toilet products,Wholesale trade,7272.984285
Manufacture of soap detergents and toilet products,Retail trade,8432.784917
Manufacture of soap detergents and toilet products,Accommodation,5702.917643
Manufacture of soap detergents and toilet products,Food and beverage service activities,5339.274826
Manufacture of soap detergents and toilet products,Transport via railways,23.00722051
Manufacture of soap detergents and toilet products,Other passenger land transport,2893.429588
Manufacture of soap detergents and toilet products,Freight transport by road,1423.325592
Manufacture of soap detergents and toilet products,Transport via pipeline,0.275898674
Manufacture of soap detergents and toilet products,Water transport,68.43937701
Manufacture of soap detergents and toilet products,Air transport,120.9080688
Manufacture of soap detergents and toilet products,Warehousing,3.328210458
Manufacture of soap detergents and toilet products,Support activities for land transport,6.922098884
Manufacture of soap detergents and toilet products,Other support activities for transport,195.006674
Manufacture of soap detergents and toilet products,Postal and courier activities,3.720685335
Manufacture of soap detergents and toilet products,Wireless telecommunications,976.2714306
Manufacture of soap detergents and toilet products,Wired telecommunications,262.9277179
Manufacture of soap detergents and toilet products,Other telecommunications activities,339.3944255
Manufacture of soap detergents and toilet products,Information services activities,976.8814669
Manufacture of soap detergents and toilet products,Publishing activities,564.1579366
Manufacture of soap detergents and toilet products,Banking monetary intermediation,218.0982999
Manufacture of soap detergents and toilet products,Insurance activities,113.2439122
Manufacture of soap detergents and toilet products,Auxiliary financial activities,220.0628202
Manufacture of soap detergents and toilet products,Real estate activities,3163.716814
Manufacture of soap detergents and toilet products,Housing services,0
Manufacture of soap detergents and toilet products,Legal and accounting services,1690.591243
Manufacture of soap detergents and toilet products,Architectural and engineering services,6841.379784
Manufacture of soap detergents and toilet products,Other professional activities,5002.289773
Manufacture of soap detergents and toilet products,Rental and leasing activities,4764.493153
Manufacture of soap detergents and toilet products,Business support activities,23627.72998
Manufacture of soap detergents and toilet products,Public administration,3844.717639
Manufacture of soap detergents and toilet products,Public education,3379.156015
Manufacture of soap detergents and toilet products,Private education,2897.688026
Manufacture of soap detergents and toilet products,Public health activities,4790.792857
Manufacture of soap detergents and toilet products,Private health activities,3237.63026
Manufacture of soap detergents and toilet products,Activities of organizations,256.9363927
Manufacture of soap detergents and toilet products,Artistic and entertainment activities,3193.800222
Manufacture of soap detergents and toilet products,Other personal services,4383.541536
Manufacture of other chemical products,Cultivation of annual crops,747.1952473
Manufacture of other chemical products,Cultivation of vegetables,336.4470291
Manufacture of other chemical products,Cultivation of grapes,539.6204053
Manufacture of other chemical products,Cultivation of other fruits,792.9902034
Manufacture of other chemical products,Cattle bredding,259.6989388
Manufacture of other chemical products,Pigs breeding,37.78164022
Manufacture of other chemical products,Chicken breeding,46.20216692
Manufacture of other chemical products,Breeding of other animals,25.90634793
Manufacture of other chemical products,Farming services,160.8205229
Manufacture of other chemical products,Forestry,583.7474642
Manufacture of other chemical products,Aquaculture,318.4422507
Manufacture of other chemical products,Extractive fishing,64.94899379
Manufacture of other chemical products,Coal mining,2209.560996
Manufacture of other chemical products,Crude oil and gas mining,9.727098048
Manufacture of other chemical products,Copper mining,127363.1965
Manufacture of other chemical products,Iron mining,581.2338676
Manufacture of other chemical products,Mining of other metals,2217.160986
Manufacture of other chemical products,Other mining activities and mining services,446.4586469
Manufacture of other chemical products,Processing and preserving of meat,1740.446977
Manufacture of other chemical products,Processing and preserving of fishmeal and fish oil,27.82243564
Manufacture of other chemical products,Processing and preserving of fish,147.2380743
Manufacture of other chemical products,Processing and preserving of fruits and vegetables,2636.729724
Manufacture of other chemical products,Manufacture of vegetable and animal oils,1662.758366
Manufacture of other chemical products,Manufacture of dairy products,1853.664969
Manufacture of other chemical products,Manufacture of grain mill products,1230.283154
Manufacture of other chemical products,Manufacture of prepared animal feeds,11171.44983
Manufacture of other chemical products,Manufacture of bakery products,426.5918582
Manufacture of other chemical products,Manufacture of macaroni noodles and other related products,16.21611357
Manufacture of other chemical products,Manufacture of other food products,3181.815379
Manufacture of other chemical products,Distilling rectifying and blending of spirits,98.86341575
Manufacture of other chemical products,Manufacture of wines,8460.922968
Manufacture of other chemical products,Manufacture of beers,746.0552257
Manufacture of other chemical products,Manufacture of soft drinks,413.6528865
Manufacture of other chemical products,Manufacture of tobacco products,22.53446729
Manufacture of other chemical products,Manufacture of textile products,7539.734597
Manufacture of other chemical products,Manufacture of wearing apparel,259.9072276
Manufacture of other chemical products,Manufacture of leather products,1357.888129
Manufacture of other chemical products,Manufacture of footwear,674.3776569
Manufacture of other chemical products,Sawmilling and planing of wood,3020.37652
Manufacture of other chemical products,Manufacture of wood products,12253.71519
Manufacture of other chemical products,Manufacture of pulp and paper,6130.720856
Manufacture of other chemical products,Manufacture of containers of paper,2930.906802
Manufacture of other chemical products,Manufacture of other paper products,4931.775686
Manufacture of other chemical products,Printing,2223.229036
Manufacture of other chemical products,Manufacture of refined petroleum products,5944.31019
Manufacture of other chemical products,Manufacture of basic chemicals,1106.266577
Manufacture of other chemical products,Manufacture of paints,4859.329489
Manufacture of other chemical products,Manufacture of pharmaceutical products,366.2601702
Manufacture of other chemical products,Manufacture of soap detergents and toilet products,22081.97648
Manufacture of other chemical products,Manufacture of other chemical products,8149.259896
Manufacture of other chemical products,Manufacture of rubber products,200.684098
Manufacture of other chemical products,Manufacture of plastic products,2988.098902
Manufacture of other chemical products,Manufacture of glass and glass products,172.0718219
Manufacture of other chemical products,Manufacture of cement,272.0499019
Manufacture of other chemical products,Manufacture of concrete and concrete products,17167.57436
Manufacture of other chemical products,Manufacture of basic iron and steel,918.3180343
Manufacture of other chemical products,Manufacture of other basic metals,296.4254728
Manufacture of other chemical products,Manufacture of fabricated metal products,897.0158365
Manufacture of other chemical products,Manufacture of industrial and domestic machinery and equipment,2605.305132
Manufacture of other chemical products,Manufacture of electric and electronic machinery and equipment,1372.135219
Manufacture of other chemical products,Manufacture of transport equipment,340.0489937
Manufacture of other chemical products,Manufacture of furniture,6746.846871
Manufacture of other chemical products,Repair and installation of machinery and other manufacturing,1835.296614
Manufacture of other chemical products,Electric power generation,331.7318438
Manufacture of other chemical products,Transmission of electric power,0.905586857
Manufacture of other chemical products,Distribution of electric power,7.577871828
Manufacture of other chemical products,Gas and steam manufacture and supply,137.6633253
Manufacture of other chemical products,Water collection treatment and supply,138.4078086
Manufacture of other chemical products,Waste collection and recycling activities,51.79370792
Manufacture of other chemical products,Construction of residential buildings,2501.258082
Manufacture of other chemical products,Construction of non-residential buildings,1986.959607
Manufacture of other chemical products,Civil engineering,38501.82266
Manufacture of other chemical products,Specialized construction activities,26756.24134
Manufacture of other chemical products,Wholesale and retail trade of motor vehicles,639.0026359
Manufacture of other chemical products,Wholesale trade,3555.671559
Manufacture of other chemical products,Retail trade,9764.517857
Manufacture of other chemical products,Accommodation,147.402221
Manufacture of other chemical products,Food and beverage service activities,879.5983095
Manufacture of other chemical products,Transport via railways,15.12257416
Manufacture of other chemical products,Other passenger land transport,303.1528424
Manufacture of other chemical products,Freight transport by road,394.5254331
Manufacture of other chemical products,Transport via pipeline,0.118214538
Manufacture of other chemical products,Water transport,32.80811545
Manufacture of other chemical products,Air transport,137.5141599
Manufacture of other chemical products,Warehousing,21.08141632
Manufacture of other chemical products,Support activities for land transport,35.93562891
Manufacture of other chemical products,Other support activities for transport,121.3252743
Manufacture of other chemical products,Postal and courier activities,5.181919412
Manufacture of other chemical products,Wireless telecommunications,922.4690361
Manufacture of other chemical products,Wired telecommunications,243.0540334
Manufacture of other chemical products,Other telecommunications activities,398.7238212
Manufacture of other chemical products,Information services activities,427.2843727
Manufacture of other chemical products,Publishing activities,184.0741409
Manufacture of other chemical products,Banking monetary intermediation,482.5244172
Manufacture of other chemical products,Insurance activities,802.9825279
Manufacture of other chemical products,Auxiliary financial activities,670.2333669
Manufacture of other chemical products,Real estate activities,257.4794658
Manufacture of other chemical products,Housing services,0
Manufacture of other chemical products,Legal and accounting services,141.2581224
Manufacture of other chemical products,Architectural and engineering services,4810.796056
Manufacture of other chemical products,Other professional activities,867.3659254
Manufacture of other chemical products,Rental and leasing activities,232.1099365
Manufacture of other chemical products,Business support activities,1052.253431
Manufacture of other chemical products,Public administration,3059.326201
Manufacture of other chemical products,Public education,7714.012688
Manufacture of other chemical products,Private education,2987.070034
Manufacture of other chemical products,Public health activities,17046.50787
Manufacture of other chemical products,Private health activities,4004.894765
Manufacture of other chemical products,Activities of organizations,1171.34314
Manufacture of other chemical products,Artistic and entertainment activities,160.6581319
Manufacture of other chemical products,Other personal services,141.9833458
Manufacture of rubber products,Cultivation of annual crops,184.4643131
Manufacture of rubber products,Cultivation of vegetables,46.36592
Manufacture of rubber products,Cultivation of grapes,173.2258652
Manufacture of rubber products,Cultivation of other fruits,244.8043857
Manufacture of rubber products,Cattle bredding,374.1397743
Manufacture of rubber products,Pigs breeding,18.54730759
Manufacture of rubber products,Chicken breeding,21.40479229
Manufacture of rubber products,Breeding of other animals,11.71183258
Manufacture of rubber products,Farming services,68.7747855
Manufacture of rubber products,Forestry,67.44939679
Manufacture of rubber products,Aquaculture,78.79342002
Manufacture of rubber products,Extractive fishing,413.8255101
Manufacture of rubber products,Coal mining,59.37225279
Manufacture of rubber products,Crude oil and gas mining,14.38830856
Manufacture of rubber products,Copper mining,3624.89485
Manufacture of rubber products,Iron mining,209.7123899
Manufacture of rubber products,Mining of other metals,708.7322699
Manufacture of rubber products,Other mining activities and mining services,268.9099472
Manufacture of rubber products,Processing and preserving of meat,157.2200391
Manufacture of rubber products,Processing and preserving of fishmeal and fish oil,18.89712056
Manufacture of rubber products,Processing and preserving of fish,105.2200943
Manufacture of rubber products,Processing and preserving of fruits and vegetables,78.29479815
Manufacture of rubber products,Manufacture of vegetable and animal oils,26.30348159
Manufacture of rubber products,Manufacture of dairy products,97.84257083
Manufacture of rubber products,Manufacture of grain mill products,45.59736917
Manufacture of rubber products,Manufacture of prepared animal feeds,167.6582947
Manufacture of rubber products,Manufacture of bakery products,122.7918554
Manufacture of rubber products,Manufacture of macaroni noodles and other related products,7.73090561
Manufacture of rubber products,Manufacture of other food products,124.0172624
Manufacture of rubber products,Distilling rectifying and blending of spirits,6.302210331
Manufacture of rubber products,Manufacture of wines,96.33063863
Manufacture of rubber products,Manufacture of beers,36.47129943
Manufacture of rubber products,Manufacture of soft drinks,119.4981569
Manufacture of rubber products,Manufacture of tobacco products,3.332381061
Manufacture of rubber products,Manufacture of textile products,218.9918419
Manufacture of rubber products,Manufacture of wearing apparel,1320.205802
Manufacture of rubber products,Manufacture of leather products,6.035041942
Manufacture of rubber products,Manufacture of footwear,16.83976014
Manufacture of rubber products,Sawmilling and planing of wood,46.85166799
Manufacture of rubber products,Manufacture of wood products,173.1291706
Manufacture of rubber products,Manufacture of pulp and paper,501.3231141
Manufacture of rubber products,Manufacture of containers of paper,107.2171486
Manufacture of rubber products,Manufacture of other paper products,265.4219849
Manufacture of rubber products,Printing,1141.540123
Manufacture of rubber products,Manufacture of refined petroleum products,346.7864924
Manufacture of rubber products,Manufacture of basic chemicals,862.000504
Manufacture of rubber products,Manufacture of paints,27.7543658
Manufacture of rubber products,Manufacture of pharmaceutical products,90.52405106
Manufacture of rubber products,Manufacture of soap detergents and toilet products,161.1514105
Manufacture of rubber products,Manufacture of other chemical products,104.3745884
Manufacture of rubber products,Manufacture of rubber products,5620.682023
Manufacture of rubber products,Manufacture of plastic products,448.3731061
Manufacture of rubber products,Manufacture of glass and glass products,85.14663738
Manufacture of rubber products,Manufacture of cement,90.68461862
Manufacture of rubber products,Manufacture of concrete and concrete products,825.1703913
Manufacture of rubber products,Manufacture of basic iron and steel,491.85165
Manufacture of rubber products,Manufacture of other basic metals,275.2233826
Manufacture of rubber products,Manufacture of fabricated metal products,696.8503901
Manufacture of rubber products,Manufacture of industrial and domestic machinery and equipment,14003.40252
Manufacture of rubber products,Manufacture of electric and electronic machinery and equipment,319.7755696
Manufacture of rubber products,Manufacture of transport equipment,404.4756211
Manufacture of rubber products,Manufacture of furniture,160.4520996
Manufacture of rubber products,Repair and installation of machinery and other manufacturing,87.66811085
Manufacture of rubber products,Electric power generation,782.3638383
Manufacture of rubber products,Transmission of electric power,76.65539759
Manufacture of rubber products,Distribution of electric power,282.5263506
Manufacture of rubber products,Gas and steam manufacture and supply,47.23247383
Manufacture of rubber products,Water collection treatment and supply,25.78052867
Manufacture of rubber products,Waste collection and recycling activities,537.5370189
Manufacture of rubber products,Construction of residential buildings,1006.813279
Manufacture of rubber products,Construction of non-residential buildings,412.1308913
Manufacture of rubber products,Civil engineering,5075.574269
Manufacture of rubber products,Specialized construction activities,3078.959508
Manufacture of rubber products,Wholesale and retail trade of motor vehicles,350.2529236
Manufacture of rubber products,Wholesale trade,1528.1397
Manufacture of rubber products,Retail trade,2825.054209
Manufacture of rubber products,Accommodation,91.58546199
Manufacture of rubber products,Food and beverage service activities,429.9374516
Manufacture of rubber products,Transport via railways,88.99845708
Manufacture of rubber products,Other passenger land transport,596.6553969
Manufacture of rubber products,Freight transport by road,4923.303894
Manufacture of rubber products,Transport via pipeline,42.93307362
Manufacture of rubber products,Water transport,89.31357034
Manufacture of rubber products,Air transport,1107.558666
Manufacture of rubber products,Warehousing,11.73005219
Manufacture of rubber products,Support activities for land transport,22.4781258
Manufacture of rubber products,Other support activities for transport,278.5819136
Manufacture of rubber products,Postal and courier activities,11.75968146
Manufacture of rubber products,Wireless telecommunications,668.865831
Manufacture of rubber products,Wired telecommunications,430.9761794
Manufacture of rubber products,Other telecommunications activities,566.0765046
Manufacture of rubber products,Information services activities,166.6481359
Manufacture of rubber products,Publishing activities,85.75397826
Manufacture of rubber products,Banking monetary intermediation,154.7492363
Manufacture of rubber products,Insurance activities,143.51611
Manufacture of rubber products,Auxiliary financial activities,30636.57591
Manufacture of rubber products,Real estate activities,171.9185161
Manufacture of rubber products,Housing services,0
Manufacture of rubber products,Legal and accounting services,23.02802425
Manufacture of rubber products,Architectural and engineering services,398.1298192
Manufacture of rubber products,Other professional activities,211.561331
Manufacture of rubber products,Rental and leasing activities,968.6814004
Manufacture of rubber products,Business support activities,3752.07591
Manufacture of rubber products,Public administration,5441.465874
Manufacture of rubber products,Public education,117.4524428
Manufacture of rubber products,Private education,268.7157787
Manufacture of rubber products,Public health activities,652.4468553
Manufacture of rubber products,Private health activities,2878.520729
Manufacture of rubber products,Activities of organizations,234.8018766
Manufacture of rubber products,Artistic and entertainment activities,121.3288028
Manufacture of rubber products,Other personal services,139.7256945
Manufacture of plastic products,Cultivation of annual crops,3151.984733
Manufacture of plastic products,Cultivation of vegetables,1671.642347
Manufacture of plastic products,Cultivation of grapes,4140.220928
Manufacture of plastic products,Cultivation of other fruits,8673.881525
Manufacture of plastic products,Cattle bredding,772.3955024
Manufacture of plastic products,Pigs breeding,496.9141544
Manufacture of plastic products,Chicken breeding,1219.422095
Manufacture of plastic products,Breeding of other animals,156.3437983
Manufacture of plastic products,Farming services,848.5770614
Manufacture of plastic products,Forestry,465.8701424
Manufacture of plastic products,Aquaculture,14985.96685
Manufacture of plastic products,Extractive fishing,2093.373814
Manufacture of plastic products,Coal mining,37.59353342
Manufacture of plastic products,Crude oil and gas mining,8.740541755
Manufacture of plastic products,Copper mining,4402.962848
Manufacture of plastic products,Iron mining,135.5918948
Manufacture of plastic products,Mining of other metals,301.0080049
Manufacture of plastic products,Other mining activities and mining services,629.7403242
Manufacture of plastic products,Processing and preserving of meat,17836.71999
Manufacture of plastic products,Processing and preserving of fishmeal and fish oil,1661.744077
Manufacture of plastic products,Processing and preserving of fish,12096.08338
Manufacture of plastic products,Processing and preserving of fruits and vegetables,8901.124486
Manufacture of plastic products,Manufacture of vegetable and animal oils,3914.193255
Manufacture of plastic products,Manufacture of dairy products,33450.0886
Manufacture of plastic products,Manufacture of grain mill products,7036.662203
Manufacture of plastic products,Manufacture of prepared animal feeds,22754.82453
Manufacture of plastic products,Manufacture of bakery products,3273.552
Manufacture of plastic products,Manufacture of macaroni noodles and other related products,4240.269502
Manufacture of plastic products,Manufacture of other food products,56142.07492
Manufacture of plastic products,Distilling rectifying and blending of spirits,208.8757962
Manufacture of plastic products,Manufacture of wines,10918.24331
Manufacture of plastic products,Manufacture of beers,14508.05798
Manufacture of plastic products,Manufacture of soft drinks,86188.26817
Manufacture of plastic products,Manufacture of tobacco products,9154.816538
Manufacture of plastic products,Manufacture of textile products,532.6005221
Manufacture of plastic products,Manufacture of wearing apparel,1657.148488
Manufacture of plastic products,Manufacture of leather products,308.7962727
Manufacture of plastic products,Manufacture of footwear,195.0250453
Manufacture of plastic products,Sawmilling and planing of wood,391.0788508
Manufacture of plastic products,Manufacture of wood products,496.8091999
Manufacture of plastic products,Manufacture of pulp and paper,792.3523207
Manufacture of plastic products,Manufacture of containers of paper,10236.36732
Manufacture of plastic products,Manufacture of other paper products,12236.39149
Manufacture of plastic products,Printing,190.7638392
Manufacture of plastic products,Manufacture of refined petroleum products,7934.536387
Manufacture of plastic products,Manufacture of basic chemicals,16021.05415
Manufacture of plastic products,Manufacture of paints,4541.733229
Manufacture of plastic products,Manufacture of pharmaceutical products,22592.2571
Manufacture of plastic products,Manufacture of soap detergents and toilet products,22914.50431
Manufacture of plastic products,Manufacture of other chemical products,2391.666635
Manufacture of plastic products,Manufacture of rubber products,741.9223312
Manufacture of plastic products,Manufacture of plastic products,50392.51164
Manufacture of plastic products,Manufacture of glass and glass products,345.2150845
Manufacture of plastic products,Manufacture of cement,893.1957374
Manufacture of plastic products,Manufacture of concrete and concrete products,2364.606063
Manufacture of plastic products,Manufacture of basic iron and steel,1143.790767
Manufacture of plastic products,Manufacture of other basic metals,514.6779253
Manufacture of plastic products,Manufacture of fabricated metal products,1444.457719
Manufacture of plastic products,Manufacture of industrial and domestic machinery and equipment,3735.577166
Manufacture of plastic products,Manufacture of electric and electronic machinery and equipment,7216.396424
Manufacture of plastic products,Manufacture of transport equipment,303.4650043
Manufacture of plastic products,Manufacture of furniture,16726.9348
Manufacture of plastic products,Repair and installation of machinery and other manufacturing,4060.241852
Manufacture of plastic products,Electric power generation,2841.164617
Manufacture of plastic products,Transmission of electric power,5.250235872
Manufacture of plastic products,Distribution of electric power,37.53560967
Manufacture of plastic products,Gas and steam manufacture and supply,108.3392858
Manufacture of plastic products,Water collection treatment and supply,30.36315549
Manufacture of plastic products,Waste collection and recycling activities,166.2832306
Manufacture of plastic products,Construction of residential buildings,36710.02249
Manufacture of plastic products,Construction of non-residential buildings,9656.974575
Manufacture of plastic products,Civil engineering,72276.9703
Manufacture of plastic products,Specialized construction activities,166708.6893
Manufacture of plastic products,Wholesale and retail trade of motor vehicles,24997.26332
Manufacture of plastic products,Wholesale trade,26917.48585
Manufacture of plastic products,Retail trade,140231.428
Manufacture of plastic products,Accommodation,381.7362407
Manufacture of plastic products,Food and beverage service activities,12898.82413
Manufacture of plastic products,Transport via railways,34.43236204
Manufacture of plastic products,Other passenger land transport,975.9178312
Manufacture of plastic products,Freight transport by road,2095.612689
Manufacture of plastic products,Transport via pipeline,4.839171633
Manufacture of plastic products,Water transport,112.8581552
Manufacture of plastic products,Air transport,1018.538519
Manufacture of plastic products,Warehousing,20.67805655
Manufacture of plastic products,Support activities for land transport,55.47049705
Manufacture of plastic products,Other support activities for transport,578.0634484
Manufacture of plastic products,Postal and courier activities,15.35270774
Manufacture of plastic products,Wireless telecommunications,2924.91685
Manufacture of plastic products,Wired telecommunications,784.1581927
Manufacture of plastic products,Other telecommunications activities,1266.332274
Manufacture of plastic products,Information services activities,925.8272247
Manufacture of plastic products,Publishing activities,979.9851757
Manufacture of plastic products,Banking monetary intermediation,676.4263249
Manufacture of plastic products,Insurance activities,396.0735115
Manufacture of plastic products,Auxiliary financial activities,846.6637401
Manufacture of plastic products,Real estate activities,537.2152194
Manufacture of plastic products,Housing services,0
Manufacture of plastic products,Legal and accounting services,135.6298912
Manufacture of plastic products,Architectural and engineering services,639.8077988
Manufacture of plastic products,Other professional activities,1582.804156
Manufacture of plastic products,Rental and leasing activities,1324.530736
Manufacture of plastic products,Business support activities,2567.64606
Manufacture of plastic products,Public administration,2532.404917
Manufacture of plastic products,Public education,2601.996421
Manufacture of plastic products,Private education,541.3404303
Manufacture of plastic products,Public health activities,7386.708966
Manufacture of plastic products,Private health activities,5450.259982
Manufacture of plastic products,Activities of organizations,4088.483776
Manufacture of plastic products,Artistic and entertainment activities,1531.83769
Manufacture of plastic products,Other personal services,3058.745055
Manufacture of glass and glass products,Cultivation of annual crops,42.15093936
Manufacture of glass and glass products,Cultivation of vegetables,12.7404906
Manufacture of glass and glass products,Cultivation of grapes,22.21470928
Manufacture of glass and glass products,Cultivation of other fruits,52.15109092
Manufacture of glass and glass products,Cattle bredding,26.66172339
Manufacture of glass and glass products,Pigs breeding,4.760393057
Manufacture of glass and glass products,Chicken breeding,6.431110085
Manufacture of glass and glass products,Breeding of other animals,2.49289948
Manufacture of glass and glass products,Farming services,14.15710991
Manufacture of glass and glass products,Forestry,13.36157622
Manufacture of glass and glass products,Aquaculture,15.69205942
Manufacture of glass and glass products,Extractive fishing,6.768557346
Manufacture of glass and glass products,Coal mining,1.521514026
Manufacture of glass and glass products,Crude oil and gas mining,0.374405182
Manufacture of glass and glass products,Copper mining,1725.537502
Manufacture of glass and glass products,Iron mining,46.67244979
Manufacture of glass and glass products,Mining of other metals,35.88164466
Manufacture of glass and glass products,Other mining activities and mining services,31.6085978
Manufacture of glass and glass products,Processing and preserving of meat,56.3383161
Manufacture of glass and glass products,Processing and preserving of fishmeal and fish oil,6.293925467
Manufacture of glass and glass products,Processing and preserving of fish,44.7755047
Manufacture of glass and glass products,Processing and preserving of fruits and vegetables,984.5663119
Manufacture of glass and glass products,Manufacture of vegetable and animal oils,1333.565952
Manufacture of glass and glass products,Manufacture of dairy products,40.08143169
Manufacture of glass and glass products,Manufacture of grain mill products,15.05054879
Manufacture of glass and glass products,Manufacture of prepared animal feeds,30.10220981
Manufacture of glass and glass products,Manufacture of bakery products,20.88029607
Manufacture of glass and glass products,Manufacture of macaroni noodles and other related products,2.438275917
Manufacture of glass and glass products,Manufacture of other food products,1631.481971
Manufacture of glass and glass products,Distilling rectifying and blending of spirits,16336.6904
Manufacture of glass and glass products,Manufacture of wines,107955.6659
Manufacture of glass and glass products,Manufacture of beers,11568.94976
Manufacture of glass and glass products,Manufacture of soft drinks,12000.4383
Manufacture of glass and glass products,Manufacture of tobacco products,0.583379025
Manufacture of glass and glass products,Manufacture of textile products,10.1273496
Manufacture of glass and glass products,Manufacture of wearing apparel,27.07680881
Manufacture of glass and glass products,Manufacture of leather products,0.864090092
Manufacture of glass and glass products,Manufacture of footwear,5.840691161
Manufacture of glass and glass products,Sawmilling and planing of wood,53.93007031
Manufacture of glass and glass products,Manufacture of wood products,1559.830917
Manufacture of glass and glass products,Manufacture of pulp and paper,330.9317002
Manufacture of glass and glass products,Manufacture of containers of paper,10.97233519
Manufacture of glass and glass products,Manufacture of other paper products,41.12277863
Manufacture of glass and glass products,Printing,7.216751447
Manufacture of glass and glass products,Manufacture of refined petroleum products,47.7049314
Manufacture of glass and glass products,Manufacture of basic chemicals,95.55764177
Manufacture of glass and glass products,Manufacture of paints,18.84055174
Manufacture of glass and glass products,Manufacture of pharmaceutical products,6492.230412
Manufacture of glass and glass products,Manufacture of soap detergents and toilet products,15.82606139
Manufacture of glass and glass products,Manufacture of other chemical products,380.1988746
Manufacture of glass and glass products,Manufacture of rubber products,11.09118323
Manufacture of glass and glass products,Manufacture of plastic products,1504.951704
Manufacture of glass and glass products,Manufacture of glass and glass products,8443.865256
Manufacture of glass and glass products,Manufacture of cement,41.4214925
Manufacture of glass and glass products,Manufacture of concrete and concrete products,32.66853002
Manufacture of glass and glass products,Manufacture of basic iron and steel,51.12677124
Manufacture of glass and glass products,Manufacture of other basic metals,18.95020275
Manufacture of glass and glass products,Manufacture of fabricated metal products,11045.33958
Manufacture of glass and glass products,Manufacture of industrial and domestic machinery and equipment,777.9662067
Manufacture of glass and glass products,Manufacture of electric and electronic machinery and equipment,18686.89385
Manufacture of glass and glass products,Manufacture of transport equipment,819.4862204
Manufacture of glass and glass products,Manufacture of furniture,1407.715202
Manufacture of glass and glass products,Repair and installation of machinery and other manufacturing,537.2453647
Manufacture of glass and glass products,Electric power generation,839.3870474
Manufacture of glass and glass products,Transmission of electric power,0.185618842
Manufacture of glass and glass products,Distribution of electric power,2194.418828
Manufacture of glass and glass products,Gas and steam manufacture and supply,20.99839161
Manufacture of glass and glass products,Water collection treatment and supply,1.463744836
Manufacture of glass and glass products,Waste collection and recycling activities,2.631393469
Manufacture of glass and glass products,Construction of residential buildings,7452.011481
Manufacture of glass and glass products,Construction of non-residential buildings,6513.095226
Manufacture of glass and glass products,Civil engineering,513.6906374
Manufacture of glass and glass products,Specialized construction activities,29300.34111
Manufacture of glass and glass products,Wholesale and retail trade of motor vehicles,9126.45296
Manufacture of glass and glass products,Wholesale trade,196.6078932
Manufacture of glass and glass products,Retail trade,338.0901269
Manufacture of glass and glass products,Accommodation,186.7486041
Manufacture of glass and glass products,Food and beverage service activities,2431.580786
Manufacture of glass and glass products,Transport via railways,1.175676899
Manufacture of glass and glass products,Other passenger land transport,31.33718464
Manufacture of glass and glass products,Freight transport by road,73.29716416
Manufacture of glass and glass products,Transport via pipeline,0.046616591
Manufacture of glass and glass products,Water transport,3.57994084
Manufacture of glass and glass products,Air transport,17.28086469
Manufacture of glass and glass products,Warehousing,0.361044457
Manufacture of glass and glass products,Support activities for land transport,1.552491549
Manufacture of glass and glass products,Other support activities for transport,22.79674507
Manufacture of glass and glass products,Postal and courier activities,0.309848358
Manufacture of glass and glass products,Wireless telecommunications,95.98134103
Manufacture of glass and glass products,Wired telecommunications,34.62503773
Manufacture of glass and glass products,Other telecommunications activities,33.96560777
Manufacture of glass and glass products,Information services activities,69.5503828
Manufacture of glass and glass products,Publishing activities,26.83009979
Manufacture of glass and glass products,Banking monetary intermediation,59.74868537
Manufacture of glass and glass products,Insurance activities,14.66705015
Manufacture of glass and glass products,Auxiliary financial activities,1889.730855
Manufacture of glass and glass products,Real estate activities,483.228075
Manufacture of glass and glass products,Housing services,0
Manufacture of glass and glass products,Legal and accounting services,3.776710055
Manufacture of glass and glass products,Architectural and engineering services,557.3667446
Manufacture of glass and glass products,Other professional activities,50.04622405
Manufacture of glass and glass products,Rental and leasing activities,20.20540245
Manufacture of glass and glass products,Business support activities,102.0598348
Manufacture of glass and glass products,Public administration,48.16439877
Manufacture of glass and glass products,Public education,2625.990371
Manufacture of glass and glass products,Private education,550.5182921
Manufacture of glass and glass products,Public health activities,112.1979427
Manufacture of glass and glass products,Private health activities,92.24175647
Manufacture of glass and glass products,Activities of organizations,20.95523883
Manufacture of glass and glass products,Artistic and entertainment activities,15.70194759
Manufacture of glass and glass products,Other personal services,13.80824922
Manufacture of cement,Cultivation of annual crops,184.0966079
Manufacture of cement,Cultivation of vegetables,90.72953657
Manufacture of cement,Cultivation of grapes,191.6933152
Manufacture of cement,Cultivation of other fruits,216.9532258
Manufacture of cement,Cattle bredding,57.59585647
Manufacture of cement,Pigs breeding,6.923106178
Manufacture of cement,Chicken breeding,8.392866862
Manufacture of cement,Breeding of other animals,5.737088377
Manufacture of cement,Farming services,2546.608744
Manufacture of cement,Forestry,29.62414096
Manufacture of cement,Aquaculture,105.3173059
Manufacture of cement,Extractive fishing,13.37156468
Manufacture of cement,Coal mining,2.07968459
Manufacture of cement,Crude oil and gas mining,3.977615145
Manufacture of cement,Copper mining,48821.41409
Manufacture of cement,Iron mining,583.6598413
Manufacture of cement,Mining of other metals,6196.343393
Manufacture of cement,Other mining activities and mining services,29.38480183
Manufacture of cement,Processing and preserving of meat,98.98781057
Manufacture of cement,Processing and preserving of fishmeal and fish oil,5.087644338
Manufacture of cement,Processing and preserving of fish,17.09883182
Manufacture of cement,Processing and preserving of fruits and vegetables,63.87956834
Manufacture of cement,Manufacture of vegetable and animal oils,10.10532725
Manufacture of cement,Manufacture of dairy products,58.13939004
Manufacture of cement,Manufacture of grain mill products,17.86473124
Manufacture of cement,Manufacture of prepared animal feeds,111.648535
Manufacture of cement,Manufacture of bakery products,58.29518149
Manufacture of cement,Manufacture of macaroni noodles and other related products,2.973317218
Manufacture of cement,Manufacture of other food products,156.4903129
Manufacture of cement,Distilling rectifying and blending of spirits,3.080512737
Manufacture of cement,Manufacture of wines,45.13111543
Manufacture of cement,Manufacture of beers,15.49440672
Manufacture of cement,Manufacture of soft drinks,68.57300532
Manufacture of cement,Manufacture of tobacco products,1.859723079
Manufacture of cement,Manufacture of textile products,71.21381235
Manufacture of cement,Manufacture of wearing apparel,56.90368163
Manufacture of cement,Manufacture of leather products,22.85478807
Manufacture of cement,Manufacture of footwear,18.0496549
Manufacture of cement,Sawmilling and planing of wood,21.39986578
Manufacture of cement,Manufacture of wood products,175.340282
Manufacture of cement,Manufacture of pulp and paper,350.9059033
Manufacture of cement,Manufacture of containers of paper,102.2121902
Manufacture of cement,Manufacture of other paper products,52.06068315
Manufacture of cement,Printing,20.79758394
Manufacture of cement,Manufacture of refined petroleum products,23.52665245
Manufacture of cement,Manufacture of basic chemicals,426.6112198
Manufacture of cement,Manufacture of paints,35.36974069
Manufacture of cement,Manufacture of pharmaceutical products,87.11333574
Manufacture of cement,Manufacture of soap detergents and toilet products,78.60477803
Manufacture of cement,Manufacture of other chemical products,1521.587287
Manufacture of cement,Manufacture of rubber products,76.86060533
Manufacture of cement,Manufacture of plastic products,479.4494417
Manufacture of cement,Manufacture of glass and glass products,50.28719661
Manufacture of cement,Manufacture of cement,4881.629581
Manufacture of cement,Manufacture of concrete and concrete products,196099.7666
Manufacture of cement,Manufacture of basic iron and steel,1720.387541
Manufacture of cement,Manufacture of other basic metals,28.95432266
Manufacture of cement,Manufacture of fabricated metal products,153.1298135
Manufacture of cement,Manufacture of industrial and domestic machinery and equipment,541.392678
Manufacture of cement,Manufacture of electric and electronic machinery and equipment,253.4477923
Manufacture of cement,Manufacture of transport equipment,20.41535376
Manufacture of cement,Manufacture of furniture,65.15326287
Manufacture of cement,Repair and installation of machinery and other manufacturing,56.60066715
Manufacture of cement,Electric power generation,28.6146042
Manufacture of cement,Transmission of electric power,0.651213546
Manufacture of cement,Distribution of electric power,11.04211293
Manufacture of cement,Gas and steam manufacture and supply,9.62791535
Manufacture of cement,Water collection treatment and supply,45.47083812
Manufacture of cement,Waste collection and recycling activities,14.09180594
Manufacture of cement,Construction of residential buildings,54125.39115
Manufacture of cement,Construction of non-residential buildings,24394.6522
Manufacture of cement,Civil engineering,117111.0026
Manufacture of cement,Specialized construction activities,76456.73832
Manufacture of cement,Wholesale and retail trade of motor vehicles,119.508436
Manufacture of cement,Wholesale trade,448.5927772
Manufacture of cement,Retail trade,705.5659285
Manufacture of cement,Accommodation,23.30061851
Manufacture of cement,Food and beverage service activities,121.150808
Manufacture of cement,Transport via railways,2.986796548
Manufacture of cement,Other passenger land transport,56.0126616
Manufacture of cement,Freight transport by road,2421.89209
Manufacture of cement,Transport via pipeline,0.16290296
Manufacture of cement,Water transport,7.23158103
Manufacture of cement,Air transport,48.28958727
Manufacture of cement,Warehousing,1.649776999
Manufacture of cement,Support activities for land transport,6.504007481
Manufacture of cement,Other support activities for transport,41.88143667
Manufacture of cement,Postal and courier activities,0.66123835
Manufacture of cement,Wireless telecommunications,179.9239694
Manufacture of cement,Wired telecommunications,86.57283056
Manufacture of cement,Other telecommunications activities,68.87497347
Manufacture of cement,Information services activities,254.4825795
Manufacture of cement,Publishing activities,30.70263719
Manufacture of cement,Banking monetary intermediation,221.9986314
Manufacture of cement,Insurance activities,56.68873056
Manufacture of cement,Auxiliary financial activities,81.3519848
Manufacture of cement,Real estate activities,29.61777402
Manufacture of cement,Housing services,0
Manufacture of cement,Legal and accounting services,16.30014378
Manufacture of cement,Architectural and engineering services,70.49217661
Manufacture of cement,Other professional activities,179.7274411
Manufacture of cement,Rental and leasing activities,42.30500228
Manufacture of cement,Business support activities,187.261004
Manufacture of cement,Public administration,1024.636381
Manufacture of cement,Public education,14.651184
Manufacture of cement,Private education,38.25415417
Manufacture of cement,Public health activities,199.9786979
Manufacture of cement,Private health activities,220.9997377
Manufacture of cement,Activities of organizations,30.17828256
Manufacture of cement,Artistic and entertainment activities,32.73208637
Manufacture of cement,Other personal services,25.48549242
Manufacture of concrete and concrete products,Cultivation of annual crops,97.60233029
Manufacture of concrete and concrete products,Cultivation of vegetables,513.1912157
Manufacture of concrete and concrete products,Cultivation of grapes,73.35567952
Manufacture of concrete and concrete products,Cultivation of other fruits,135.8094565
Manufacture of concrete and concrete products,Cattle bredding,61.33207508
Manufacture of concrete and concrete products,Pigs breeding,96.74570539
Manufacture of concrete and concrete products,Chicken breeding,353.9415195
Manufacture of concrete and concrete products,Breeding of other animals,6.212116412
Manufacture of concrete and concrete products,Farming services,54.35470644
Manufacture of concrete and concrete products,Forestry,35.15347352
Manufacture of concrete and concrete products,Aquaculture,634.2267661
Manufacture of concrete and concrete products,Extractive fishing,28.12893786
Manufacture of concrete and concrete products,Coal mining,67.09046714
Manufacture of concrete and concrete products,Crude oil and gas mining,1.004049045
Manufacture of concrete and concrete products,Copper mining,6267.487078
Manufacture of concrete and concrete products,Iron mining,28.50324584
Manufacture of concrete and concrete products,Mining of other metals,121.6068734
Manufacture of concrete and concrete products,Other mining activities and mining services,20.44034881
Manufacture of concrete and concrete products,Processing and preserving of meat,245.9887496
Manufacture of concrete and concrete products,Processing and preserving of fishmeal and fish oil,13.33897118
Manufacture of concrete and concrete products,Processing and preserving of fish,128.4515063
Manufacture of concrete and concrete products,Processing and preserving of fruits and vegetables,147.8090709
Manufacture of concrete and concrete products,Manufacture of vegetable and animal oils,69.98234149
Manufacture of concrete and concrete products,Manufacture of dairy products,219.5769709
Manufacture of concrete and concrete products,Manufacture of grain mill products,97.74323328
Manufacture of concrete and concrete products,Manufacture of prepared animal feeds,426.8345079
Manufacture of concrete and concrete products,Manufacture of bakery products,104.383123
Manufacture of concrete and concrete products,Manufacture of macaroni noodles and other related products,12.00221244
Manufacture of concrete and concrete products,Manufacture of other food products,279.6099601
Manufacture of concrete and concrete products,Distilling rectifying and blending of spirits,23.35414208
Manufacture of concrete and concrete products,Manufacture of wines,387.4859785
Manufacture of concrete and concrete products,Manufacture of beers,114.9741778
Manufacture of concrete and concrete products,Manufacture of soft drinks,316.6382406
Manufacture of concrete and concrete products,Manufacture of tobacco products,33.97683313
Manufacture of concrete and concrete products,Manufacture of textile products,269.1296712
Manufacture of concrete and concrete products,Manufacture of wearing apparel,187.3580572
Manufacture of concrete and concrete products,Manufacture of leather products,61.30186286
Manufacture of concrete and concrete products,Manufacture of footwear,37.43083813
Manufacture of concrete and concrete products,Sawmilling and planing of wood,167.6671077
Manufacture of concrete and concrete products,Manufacture of wood products,389.60642
Manufacture of concrete and concrete products,Manufacture of pulp and paper,255.526194
Manufacture of concrete and concrete products,Manufacture of containers of paper,129.9136203
Manufacture of concrete and concrete products,Manufacture of other paper products,249.6633035
Manufacture of concrete and concrete products,Printing,118.4440243
Manufacture of concrete and concrete products,Manufacture of refined petroleum products,231.2381883
Manufacture of concrete and concrete products,Manufacture of basic chemicals,120.6084741
Manufacture of concrete and concrete products,Manufacture of paints,177.9375025
Manufacture of concrete and concrete products,Manufacture of pharmaceutical products,244.4467724
Manufacture of concrete and concrete products,Manufacture of soap detergents and toilet products,696.7704353
Manufacture of concrete and concrete products,Manufacture of other chemical products,253.0903316
Manufacture of concrete and concrete products,Manufacture of rubber products,44.26099382
Manufacture of concrete and concrete products,Manufacture of plastic products,259.7138898
Manufacture of concrete and concrete products,Manufacture of glass and glass products,3486.388859
Manufacture of concrete and concrete products,Manufacture of cement,813.8265184
Manufacture of concrete and concrete products,Manufacture of concrete and concrete products,10817.17058
Manufacture of concrete and concrete products,Manufacture of basic iron and steel,186.9438714
Manufacture of concrete and concrete products,Manufacture of other basic metals,47.08846931
Manufacture of concrete and concrete products,Manufacture of fabricated metal products,3062.971828
Manufacture of concrete and concrete products,Manufacture of industrial and domestic machinery and equipment,8485.696268
Manufacture of concrete and concrete products,Manufacture of electric and electronic machinery and equipment,4926.082275
Manufacture of concrete and concrete products,Manufacture of transport equipment,72.64264313
Manufacture of concrete and concrete products,Manufacture of furniture,321.876218
Manufacture of concrete and concrete products,Repair and installation of machinery and other manufacturing,571.1641128
Manufacture of concrete and concrete products,Electric power generation,136.7658177
Manufacture of concrete and concrete products,Transmission of electric power,0.876373823
Manufacture of concrete and concrete products,Distribution of electric power,66.91651971
Manufacture of concrete and concrete products,Gas and steam manufacture and supply,136.5356814
Manufacture of concrete and concrete products,Water collection treatment and supply,10.73976925
Manufacture of concrete and concrete products,Waste collection and recycling activities,10.32605015
Manufacture of concrete and concrete products,Construction of residential buildings,140665.0443
Manufacture of concrete and concrete products,Construction of non-residential buildings,143089.9428
Manufacture of concrete and concrete products,Civil engineering,443505.2887
Manufacture of concrete and concrete products,Specialized construction activities,508251.5395
Manufacture of concrete and concrete products,Wholesale and retail trade of motor vehicles,281.4517699
Manufacture of concrete and concrete products,Wholesale trade,1177.85502
Manufacture of concrete and concrete products,Retail trade,1565.903789
Manufacture of concrete and concrete products,Accommodation,36.39095091
Manufacture of concrete and concrete products,Food and beverage service activities,208.5951466
Manufacture of concrete and concrete products,Transport via railways,9.96669535
Manufacture of concrete and concrete products,Other passenger land transport,82.81300357
Manufacture of concrete and concrete products,Freight transport by road,133.1864763
Manufacture of concrete and concrete products,Transport via pipeline,0.128842872
Manufacture of concrete and concrete products,Water transport,11.70425354
Manufacture of concrete and concrete products,Air transport,82.70411328
Manufacture of concrete and concrete products,Warehousing,20.03649952
Manufacture of concrete and concrete products,Support activities for land transport,35.55390208
Manufacture of concrete and concrete products,Other support activities for transport,67.74330286
Manufacture of concrete and concrete products,Postal and courier activities,3.174064419
Manufacture of concrete and concrete products,Wireless telecommunications,296.0200777
Manufacture of concrete and concrete products,Wired telecommunications,101.2078019
Manufacture of concrete and concrete products,Other telecommunications activities,187.5189629
Manufacture of concrete and concrete products,Information services activities,325.6320151
Manufacture of concrete and concrete products,Publishing activities,109.6806198
Manufacture of concrete and concrete products,Banking monetary intermediation,438.9549176
Manufacture of concrete and concrete products,Insurance activities,236.1697655
Manufacture of concrete and concrete products,Auxiliary financial activities,127.5071061
Manufacture of concrete and concrete products,Real estate activities,416.1873851
Manufacture of concrete and concrete products,Housing services,0
Manufacture of concrete and concrete products,Legal and accounting services,128.7575617
Manufacture of concrete and concrete products,Architectural and engineering services,397.446684
Manufacture of concrete and concrete products,Other professional activities,763.4265792
Manufacture of concrete and concrete products,Rental and leasing activities,190.1859651
Manufacture of concrete and concrete products,Business support activities,580.2659411
Manufacture of concrete and concrete products,Public administration,2469.428984
Manufacture of concrete and concrete products,Public education,5866.162874
Manufacture of concrete and concrete products,Private education,258.8028863
Manufacture of concrete and concrete products,Public health activities,1167.357774
Manufacture of concrete and concrete products,Private health activities,409.6923042
Manufacture of concrete and concrete products,Activities of organizations,66.69629795
Manufacture of concrete and concrete products,Artistic and entertainment activities,76.5203678
Manufacture of concrete and concrete products,Other personal services,37.62365891
Manufacture of basic iron and steel,Cultivation of annual crops,274.1849024
Manufacture of basic iron and steel,Cultivation of vegetables,168.2487095
Manufacture of basic iron and steel,Cultivation of grapes,338.6909135
Manufacture of basic iron and steel,Cultivation of other fruits,961.3538972
Manufacture of basic iron and steel,Cattle bredding,172.7950905
Manufacture of basic iron and steel,Pigs breeding,58.08482813
Manufacture of basic iron and steel,Chicken breeding,259.8970354
Manufacture of basic iron and steel,Breeding of other animals,65.56630943
Manufacture of basic iron and steel,Farming services,201.1233366
Manufacture of basic iron and steel,Forestry,125.9648646
Manufacture of basic iron and steel,Aquaculture,82.58862574
Manufacture of basic iron and steel,Extractive fishing,36.6866967
Manufacture of basic iron and steel,Coal mining,10.81057729
Manufacture of basic iron and steel,Crude oil and gas mining,2.141512042
Manufacture of basic iron and steel,Copper mining,50072.67956
Manufacture of basic iron and steel,Iron mining,995.6413197
Manufacture of basic iron and steel,Mining of other metals,76.11109571
Manufacture of basic iron and steel,Other mining activities and mining services,182.1830345
Manufacture of basic iron and steel,Processing and preserving of meat,121.3831619
Manufacture of basic iron and steel,Processing and preserving of fishmeal and fish oil,10.7728586
Manufacture of basic iron and steel,Processing and preserving of fish,81.57430111
Manufacture of basic iron and steel,Processing and preserving of fruits and vegetables,492.0809174
Manufacture of basic iron and steel,Manufacture of vegetable and animal oils,28.19568074
Manufacture of basic iron and steel,Manufacture of dairy products,111.6396125
Manufacture of basic iron and steel,Manufacture of grain mill products,113.0818641
Manufacture of basic iron and steel,Manufacture of prepared animal feeds,152.8964706
Manufacture of basic iron and steel,Manufacture of bakery products,98.03058829
Manufacture of basic iron and steel,Manufacture of macaroni noodles and other related products,7.980521727
Manufacture of basic iron and steel,Manufacture of other food products,445.6489041
Manufacture of basic iron and steel,Distilling rectifying and blending of spirits,48.36827291
Manufacture of basic iron and steel,Manufacture of wines,77.32180747
Manufacture of basic iron and steel,Manufacture of beers,804.7258354
Manufacture of basic iron and steel,Manufacture of soft drinks,225.3536215
Manufacture of basic iron and steel,Manufacture of tobacco products,4.737051921
Manufacture of basic iron and steel,Manufacture of textile products,57.26377647
Manufacture of basic iron and steel,Manufacture of wearing apparel,196.6872843
Manufacture of basic iron and steel,Manufacture of leather products,8.505095122
Manufacture of basic iron and steel,Manufacture of footwear,44.05895016
Manufacture of basic iron and steel,Sawmilling and planing of wood,309.3150345
Manufacture of basic iron and steel,Manufacture of wood products,79.45426332
Manufacture of basic iron and steel,Manufacture of pulp and paper,1230.500003
Manufacture of basic iron and steel,Manufacture of containers of paper,112.6238065
Manufacture of basic iron and steel,Manufacture of other paper products,89.16677368
Manufacture of basic iron and steel,Printing,35.812472
Manufacture of basic iron and steel,Manufacture of refined petroleum products,94.34573278
Manufacture of basic iron and steel,Manufacture of basic chemicals,526.3564231
Manufacture of basic iron and steel,Manufacture of paints,301.5655993
Manufacture of basic iron and steel,Manufacture of pharmaceutical products,138.425295
Manufacture of basic iron and steel,Manufacture of soap detergents and toilet products,71.01142915
Manufacture of basic iron and steel,Manufacture of other chemical products,171.02706
Manufacture of basic iron and steel,Manufacture of rubber products,130.3342214
Manufacture of basic iron and steel,Manufacture of plastic products,486.3039541
Manufacture of basic iron and steel,Manufacture of glass and glass products,211.770826
Manufacture of basic iron and steel,Manufacture of cement,1884.392844
Manufacture of basic iron and steel,Manufacture of concrete and concrete products,5138.099239
Manufacture of basic iron and steel,Manufacture of basic iron and steel,99639.34672
Manufacture of basic iron and steel,Manufacture of other basic metals,388.8576217
Manufacture of basic iron and steel,Manufacture of fabricated metal products,127493.906
Manufacture of basic iron and steel,Manufacture of industrial and domestic machinery and equipment,85595.59398
Manufacture of basic iron and steel,Manufacture of electric and electronic machinery and equipment,29862.40732
Manufacture of basic iron and steel,Manufacture of transport equipment,19386.82719
Manufacture of basic iron and steel,Manufacture of furniture,22854.97791
Manufacture of basic iron and steel,Repair and installation of machinery and other manufacturing,30434.51257
Manufacture of basic iron and steel,Electric power generation,113.6694366
Manufacture of basic iron and steel,Transmission of electric power,4.871195939
Manufacture of basic iron and steel,Distribution of electric power,28.93546969
Manufacture of basic iron and steel,Gas and steam manufacture and supply,208.9832649
Manufacture of basic iron and steel,Water collection treatment and supply,8.447753675
Manufacture of basic iron and steel,Waste collection and recycling activities,59.17536946
Manufacture of basic iron and steel,Construction of residential buildings,7468.969385
Manufacture of basic iron and steel,Construction of non-residential buildings,43218.91976
Manufacture of basic iron and steel,Civil engineering,216472.0223
Manufacture of basic iron and steel,Specialized construction activities,127686.883
Manufacture of basic iron and steel,Wholesale and retail trade of motor vehicles,319.8550316
Manufacture of basic iron and steel,Wholesale trade,1106.251782
Manufacture of basic iron and steel,Retail trade,1684.754749
Manufacture of basic iron and steel,Accommodation,64.27611334
Manufacture of basic iron and steel,Food and beverage service activities,335.3104025
Manufacture of basic iron and steel,Transport via railways,10.84100072
Manufacture of basic iron and steel,Other passenger land transport,188.233581
Manufacture of basic iron and steel,Freight transport by road,628.7737707
Manufacture of basic iron and steel,Transport via pipeline,2.59345697
Manufacture of basic iron and steel,Water transport,21.62828105
Manufacture of basic iron and steel,Air transport,69.54327405
Manufacture of basic iron and steel,Warehousing,4.346147173
Manufacture of basic iron and steel,Support activities for land transport,8.32394899
Manufacture of basic iron and steel,Other support activities for transport,203.6678934
Manufacture of basic iron and steel,Postal and courier activities,2.358446912
Manufacture of basic iron and steel,Wireless telecommunications,491.978293
Manufacture of basic iron and steel,Wired telecommunications,153.8214031
Manufacture of basic iron and steel,Other telecommunications activities,200.1836547
Manufacture of basic iron and steel,Information services activities,178.1354641
Manufacture of basic iron and steel,Publishing activities,373.1793024
Manufacture of basic iron and steel,Banking monetary intermediation,160.5416066
Manufacture of basic iron and steel,Insurance activities,54.02051382
Manufacture of basic iron and steel,Auxiliary financial activities,145.6394383
Manufacture of basic iron and steel,Real estate activities,143.5975817
Manufacture of basic iron and steel,Housing services,0
Manufacture of basic iron and steel,Legal and accounting services,28.90654959
Manufacture of basic iron and steel,Architectural and engineering services,356.8349289
Manufacture of basic iron and steel,Other professional activities,198.500304
Manufacture of basic iron and steel,Rental and leasing activities,223.1397479
Manufacture of basic iron and steel,Business support activities,1408.02704
Manufacture of basic iron and steel,Public administration,1373.412576
Manufacture of basic iron and steel,Public education,127.7015061
Manufacture of basic iron and steel,Private education,67.59720712
Manufacture of basic iron and steel,Public health activities,2190.92009
Manufacture of basic iron and steel,Private health activities,536.4400924
Manufacture of basic iron and steel,Activities of organizations,357.9620604
Manufacture of basic iron and steel,Artistic and entertainment activities,68.24430989
Manufacture of basic iron and steel,Other personal services,79.22905165
Manufacture of other basic metals,Cultivation of annual crops,72.71356267
Manufacture of other basic metals,Cultivation of vegetables,27.76693885
Manufacture of other basic metals,Cultivation of grapes,233.3422306
Manufacture of other basic metals,Cultivation of other fruits,196.9482247
Manufacture of other basic metals,Cattle bredding,182.9016149
Manufacture of other basic metals,Pigs breeding,44.97621996
Manufacture of other basic metals,Chicken breeding,81.05653789
Manufacture of other basic metals,Breeding of other animals,15.86379997
Manufacture of other basic metals,Farming services,291.6262039
Manufacture of other basic metals,Forestry,27.00671289
Manufacture of other basic metals,Aquaculture,33.06034981
Manufacture of other basic metals,Extractive fishing,194.7952168
Manufacture of other basic metals,Coal mining,1.3627575
Manufacture of other basic metals,Crude oil and gas mining,3.491268746
Manufacture of other basic metals,Copper mining,940.0785196
Manufacture of other basic metals,Iron mining,20.43086417
Manufacture of other basic metals,Mining of other metals,92.03080424
Manufacture of other basic metals,Other mining activities and mining services,22.53415154
Manufacture of other basic metals,Processing and preserving of meat,86.29274319
Manufacture of other basic metals,Processing and preserving of fishmeal and fish oil,7.03168255
Manufacture of other basic metals,Processing and preserving of fish,59.53116329
Manufacture of other basic metals,Processing and preserving of fruits and vegetables,2482.966541
Manufacture of other basic metals,Manufacture of vegetable and animal oils,10.58463251
Manufacture of other basic metals,Manufacture of dairy products,292.1471
Manufacture of other basic metals,Manufacture of grain mill products,32.05333503
Manufacture of other basic metals,Manufacture of prepared animal feeds,66.4775921
Manufacture of other basic metals,Manufacture of bakery products,60.19279082
Manufacture of other basic metals,Manufacture of macaroni noodles and other related products,2.174611624
Manufacture of other basic metals,Manufacture of other food products,206.4727019
Manufacture of other basic metals,Distilling rectifying and blending of spirits,9.488552723
Manufacture of other basic metals,Manufacture of wines,99.64694895
Manufacture of other basic metals,Manufacture of beers,495.0394106
Manufacture of other basic metals,Manufacture of soft drinks,103.2122485
Manufacture of other basic metals,Manufacture of tobacco products,7.080030044
Manufacture of other basic metals,Manufacture of textile products,89.00483091
Manufacture of other basic metals,Manufacture of wearing apparel,1652.266807
Manufacture of other basic metals,Manufacture of leather products,21.40971931
Manufacture of other basic metals,Manufacture of footwear,22.91949543
Manufacture of other basic metals,Sawmilling and planing of wood,67.29065602
Manufacture of other basic metals,Manufacture of wood products,1764.818596
Manufacture of other basic metals,Manufacture of pulp and paper,135.0603082
Manufacture of other basic metals,Manufacture of containers of paper,1433.235162
Manufacture of other basic metals,Manufacture of other paper products,82.0739815
Manufacture of other basic metals,Printing,33.74186133
Manufacture of other basic metals,Manufacture of refined petroleum products,43.48326499
Manufacture of other basic metals,Manufacture of basic chemicals,768.09994
Manufacture of other basic metals,Manufacture of paints,22.38804506
Manufacture of other basic metals,Manufacture of pharmaceutical products,605.6157957
Manufacture of other basic metals,Manufacture of soap detergents and toilet products,71.25494378
Manufacture of other basic metals,Manufacture of other chemical products,291.8158848
Manufacture of other basic metals,Manufacture of rubber products,132.6866437
Manufacture of other basic metals,Manufacture of plastic products,147.6889262
Manufacture of other basic metals,Manufacture of glass and glass products,443.6725668
Manufacture of other basic metals,Manufacture of cement,2680.308541
Manufacture of other basic metals,Manufacture of concrete and concrete products,2436.477318
Manufacture of other basic metals,Manufacture of basic iron and steel,1393.634526
Manufacture of other basic metals,Manufacture of other basic metals,25589.47184
Manufacture of other basic metals,Manufacture of fabricated metal products,215.9385583
Manufacture of other basic metals,Manufacture of industrial and domestic machinery and equipment,5477.044426
Manufacture of other basic metals,Manufacture of electric and electronic machinery and equipment,26022.13759
Manufacture of other basic metals,Manufacture of transport equipment,1044.899073
Manufacture of other basic metals,Manufacture of furniture,287.634577
Manufacture of other basic metals,Repair and installation of machinery and other manufacturing,7080.296836
Manufacture of other basic metals,Electric power generation,92.04713567
Manufacture of other basic metals,Transmission of electric power,12.17224809
Manufacture of other basic metals,Distribution of electric power,41.01588307
Manufacture of other basic metals,Gas and steam manufacture and supply,73.51364271
Manufacture of other basic metals,Water collection treatment and supply,18.12309199
Manufacture of other basic metals,Waste collection and recycling activities,545.8472311
Manufacture of other basic metals,Construction of residential buildings,3087.562687
Manufacture of other basic metals,Construction of non-residential buildings,19814.64489
Manufacture of other basic metals,Civil engineering,4475.246321
Manufacture of other basic metals,Specialized construction activities,25302.31571
Manufacture of other basic metals,Wholesale and retail trade of motor vehicles,333.7547816
Manufacture of other basic metals,Wholesale trade,1388.201238
Manufacture of other basic metals,Retail trade,631.7152725
Manufacture of other basic metals,Accommodation,109.5368919
Manufacture of other basic metals,Food and beverage service activities,100.5261494
Manufacture of other basic metals,Transport via railways,12.2888653
Manufacture of other basic metals,Other passenger land transport,76.9983278
Manufacture of other basic metals,Freight transport by road,146.8129545
Manufacture of other basic metals,Transport via pipeline,14.06633333
Manufacture of other basic metals,Water transport,13.02871899
Manufacture of other basic metals,Air transport,80.15241646
Manufacture of other basic metals,Warehousing,8.687457813
Manufacture of other basic metals,Support activities for land transport,23.79731654
Manufacture of other basic metals,Other support activities for transport,138.8457749
Manufacture of other basic metals,Postal and courier activities,4.077298529
Manufacture of other basic metals,Wireless telecommunications,250.4677908
Manufacture of other basic metals,Wired telecommunications,168.5062299
Manufacture of other basic metals,Other telecommunications activities,192.7055482
Manufacture of other basic metals,Information services activities,393.1749722
Manufacture of other basic metals,Publishing activities,553.2751853
Manufacture of other basic metals,Banking monetary intermediation,448.5836429
Manufacture of other basic metals,Insurance activities,129.8186412
Manufacture of other basic metals,Auxiliary financial activities,564.6603175
Manufacture of other basic metals,Real estate activities,207.6416585
Manufacture of other basic metals,Housing services,0
Manufacture of other basic metals,Legal and accounting services,77.11233624
Manufacture of other basic metals,Architectural and engineering services,238.8802921
Manufacture of other basic metals,Other professional activities,392.5113954
Manufacture of other basic metals,Rental and leasing activities,110.9910564
Manufacture of other basic metals,Business support activities,2931.415605
Manufacture of other basic metals,Public administration,2297.59111
Manufacture of other basic metals,Public education,370.2878721
Manufacture of other basic metals,Private education,245.289704
Manufacture of other basic metals,Public health activities,2473.942218
Manufacture of other basic metals,Private health activities,766.5078863
Manufacture of other basic metals,Activities of organizations,96.17894362
Manufacture of other basic metals,Artistic and entertainment activities,447.6395587
Manufacture of other basic metals,Other personal services,1037.839189
Manufacture of fabricated metal products,Cultivation of annual crops,3675.864353
Manufacture of fabricated metal products,Cultivation of vegetables,2688.876215
Manufacture of fabricated metal products,Cultivation of grapes,1397.833128
Manufacture of fabricated metal products,Cultivation of other fruits,2394.43189
Manufacture of fabricated metal products,Cattle bredding,2211.50427
Manufacture of fabricated metal products,Pigs breeding,784.9644289
Manufacture of fabricated metal products,Chicken breeding,1704.241047
Manufacture of fabricated metal products,Breeding of other animals,95.76586648
Manufacture of fabricated metal products,Farming services,685.1714122
Manufacture of fabricated metal products,Forestry,2987.055378
Manufacture of fabricated metal products,Aquaculture,1630.483112
Manufacture of fabricated metal products,Extractive fishing,537.0644441
Manufacture of fabricated metal products,Coal mining,293.701521
Manufacture of fabricated metal products,Crude oil and gas mining,223.107034
Manufacture of fabricated metal products,Copper mining,151014.0436
Manufacture of fabricated metal products,Iron mining,1436.707233
Manufacture of fabricated metal products,Mining of other metals,2736.824787
Manufacture of fabricated metal products,Other mining activities and mining services,2468.065205
Manufacture of fabricated metal products,Processing and preserving of meat,1028.052579
Manufacture of fabricated metal products,Processing and preserving of fishmeal and fish oil,134.0715328
Manufacture of fabricated metal products,Processing and preserving of fish,2083.304595
Manufacture of fabricated metal products,Processing and preserving of fruits and vegetables,14567.86698
Manufacture of fabricated metal products,Manufacture of vegetable and animal oils,311.8462985
Manufacture of fabricated metal products,Manufacture of dairy products,1042.148923
Manufacture of fabricated metal products,Manufacture of grain mill products,3332.821962
Manufacture of fabricated metal products,Manufacture of prepared animal feeds,1203.345842
Manufacture of fabricated metal products,Manufacture of bakery products,720.7601054
Manufacture of fabricated metal products,Manufacture of macaroni noodles and other related products,100.3610775
Manufacture of fabricated metal products,Manufacture of other food products,6449.281723
Manufacture of fabricated metal products,Distilling rectifying and blending of spirits,1762.935824
Manufacture of fabricated metal products,Manufacture of wines,2740.556392
Manufacture of fabricated metal products,Manufacture of beers,27511.41025
Manufacture of fabricated metal products,Manufacture of soft drinks,7005.350848
Manufacture of fabricated metal products,Manufacture of tobacco products,188.2791654
Manufacture of fabricated metal products,Manufacture of textile products,886.4051711
Manufacture of fabricated metal products,Manufacture of wearing apparel,2249.371258
Manufacture of fabricated metal products,Manufacture of leather products,339.0716362
Manufacture of fabricated metal products,Manufacture of footwear,1008.166149
Manufacture of fabricated metal products,Sawmilling and planing of wood,10206.48265
Manufacture of fabricated metal products,Manufacture of wood products,1488.08829
Manufacture of fabricated metal products,Manufacture of pulp and paper,1738.172012
Manufacture of fabricated metal products,Manufacture of containers of paper,1022.45845
Manufacture of fabricated metal products,Manufacture of other paper products,1231.229316
Manufacture of fabricated metal products,Printing,449.4474066
Manufacture of fabricated metal products,Manufacture of refined petroleum products,1686.107137
Manufacture of fabricated metal products,Manufacture of basic chemicals,952.1350717
Manufacture of fabricated metal products,Manufacture of paints,9667.347872
Manufacture of fabricated metal products,Manufacture of pharmaceutical products,3224.161304
Manufacture of fabricated metal products,Manufacture of soap detergents and toilet products,997.4882377
Manufacture of fabricated metal products,Manufacture of other chemical products,4743.226589
Manufacture of fabricated metal products,Manufacture of rubber products,4141.152119
Manufacture of fabricated metal products,Manufacture of plastic products,2958.817604
Manufacture of fabricated metal products,Manufacture of glass and glass products,3284.037221
Manufacture of fabricated metal products,Manufacture of cement,793.9843091
Manufacture of fabricated metal products,Manufacture of concrete and concrete products,3707.245343
Manufacture of fabricated metal products,Manufacture of basic iron and steel,3583.539618
Manufacture of fabricated metal products,Manufacture of other basic metals,2268.711874
Manufacture of fabricated metal products,Manufacture of fabricated metal products,19068.09797
Manufacture of fabricated metal products,Manufacture of industrial and domestic machinery and equipment,4359.199245
Manufacture of fabricated metal products,Manufacture of electric and electronic machinery and equipment,5135.314693
Manufacture of fabricated metal products,Manufacture of transport equipment,4251.773014
Manufacture of fabricated metal products,Manufacture of furniture,5751.616801
Manufacture of fabricated metal products,Repair and installation of machinery and other manufacturing,8000.9722
Manufacture of fabricated metal products,Electric power generation,4366.159387
Manufacture of fabricated metal products,Transmission of electric power,458.2351023
Manufacture of fabricated metal products,Distribution of electric power,1953.059521
Manufacture of fabricated metal products,Gas and steam manufacture and supply,6472.394991
Manufacture of fabricated metal products,Water collection treatment and supply,231.5387825
Manufacture of fabricated metal products,Waste collection and recycling activities,3156.11795
Manufacture of fabricated metal products,Construction of residential buildings,68197.36707
Manufacture of fabricated metal products,Construction of non-residential buildings,8011.306546
Manufacture of fabricated metal products,Civil engineering,301676.0828
Manufacture of fabricated metal products,Specialized construction activities,345839.2285
Manufacture of fabricated metal products,Wholesale and retail trade of motor vehicles,4770.378216
Manufacture of fabricated metal products,Wholesale trade,13536.43062
Manufacture of fabricated metal products,Retail trade,11366.01288
Manufacture of fabricated metal products,Accommodation,519.2568424
Manufacture of fabricated metal products,Food and beverage service activities,2476.166191
Manufacture of fabricated metal products,Transport via railways,576.3942354
Manufacture of fabricated metal products,Other passenger land transport,4271.489061
Manufacture of fabricated metal products,Freight transport by road,18577.07961
Manufacture of fabricated metal products,Transport via pipeline,412.6458129
Manufacture of fabricated metal products,Water transport,531.0123543
Manufacture of fabricated metal products,Air transport,2334.870212
Manufacture of fabricated metal products,Warehousing,164.8686808
Manufacture of fabricated metal products,Support activities for land transport,205.6312541
Manufacture of fabricated metal products,Other support activities for transport,6061.226134
Manufacture of fabricated metal products,Postal and courier activities,72.53616114
Manufacture of fabricated metal products,Wireless telecommunications,3535.383327
Manufacture of fabricated metal products,Wired telecommunications,2484.840683
Manufacture of fabricated metal products,Other telecommunications activities,3446.493323
Manufacture of fabricated metal products,Information services activities,2173.758575
Manufacture of fabricated metal products,Publishing activities,11053.91809
Manufacture of fabricated metal products,Banking monetary intermediation,1428.543564
Manufacture of fabricated metal products,Insurance activities,584.8569299
Manufacture of fabricated metal products,Auxiliary financial activities,4063.93035
Manufacture of fabricated metal products,Real estate activities,3577.193615
Manufacture of fabricated metal products,Housing services,0
Manufacture of fabricated metal products,Legal and accounting services,639.0660528
Manufacture of fabricated metal products,Architectural and engineering services,2976.812038
Manufacture of fabricated metal products,Other professional activities,2227.267183
Manufacture of fabricated metal products,Rental and leasing activities,7043.785037
Manufacture of fabricated metal products,Business support activities,5871.252431
Manufacture of fabricated metal products,Public administration,8131.435322
Manufacture of fabricated metal products,Public education,4301.785409
Manufacture of fabricated metal products,Private education,1258.235089
Manufacture of fabricated metal products,Public health activities,12338.58359
Manufacture of fabricated metal products,Private health activities,5611.802876
Manufacture of fabricated metal products,Activities of organizations,7757.364065
Manufacture of fabricated metal products,Artistic and entertainment activities,993.5114533
Manufacture of fabricated metal products,Other personal services,1188.841989
Manufacture of industrial and domestic machinery and equipment,Cultivation of annual crops,965.9412584
Manufacture of industrial and domestic machinery and equipment,Cultivation of vegetables,1996.941615
Manufacture of industrial and domestic machinery and equipment,Cultivation of grapes,2366.817626
Manufacture of industrial and domestic machinery and equipment,Cultivation of other fruits,6263.742214
Manufacture of industrial and domestic machinery and equipment,Cattle bredding,684.8369717
Manufacture of industrial and domestic machinery and equipment,Pigs breeding,108.4138254
Manufacture of industrial and domestic machinery and equipment,Chicken breeding,158.4424685
Manufacture of industrial and domestic machinery and equipment,Breeding of other animals,59.25658871
Manufacture of industrial and domestic machinery and equipment,Farming services,3377.480731
Manufacture of industrial and domestic machinery and equipment,Forestry,961.6105215
Manufacture of industrial and domestic machinery and equipment,Aquaculture,16103.77031
Manufacture of industrial and domestic machinery and equipment,Extractive fishing,1834.949248
Manufacture of industrial and domestic machinery and equipment,Coal mining,1591.622976
Manufacture of industrial and domestic machinery and equipment,Crude oil and gas mining,2370.465646
Manufacture of industrial and domestic machinery and equipment,Copper mining,127690.4694
Manufacture of industrial and domestic machinery and equipment,Iron mining,6248.798308
Manufacture of industrial and domestic machinery and equipment,Mining of other metals,10177.78392
Manufacture of industrial and domestic machinery and equipment,Other mining activities and mining services,11809.86287
Manufacture of industrial and domestic machinery and equipment,Processing and preserving of meat,1759.344863
Manufacture of industrial and domestic machinery and equipment,Processing and preserving of fishmeal and fish oil,177.4395142
Manufacture of industrial and domestic machinery and equipment,Processing and preserving of fish,867.0080819
Manufacture of industrial and domestic machinery and equipment,Processing and preserving of fruits and vegetables,959.0912447
Manufacture of industrial and domestic machinery and equipment,Manufacture of vegetable and animal oils,1832.491816
Manufacture of industrial and domestic machinery and equipment,Manufacture of dairy products,525.7457687
Manufacture of industrial and domestic machinery and equipment,Manufacture of grain mill products,508.9512112
Manufacture of industrial and domestic machinery and equipment,Manufacture of prepared animal feeds,1202.287015
Manufacture of industrial and domestic machinery and equipment,Manufacture of bakery products,1014.542031
Manufacture of industrial and domestic machinery and equipment,Manufacture of macaroni noodles and other related products,97.29440994
Manufacture of industrial and domestic machinery and equipment,Manufacture of other food products,2105.698862
Manufacture of industrial and domestic machinery and equipment,Distilling rectifying and blending of spirits,81.08497789
Manufacture of industrial and domestic machinery and equipment,Manufacture of wines,873.7335189
Manufacture of industrial and domestic machinery and equipment,Manufacture of beers,2588.856495
Manufacture of industrial and domestic machinery and equipment,Manufacture of soft drinks,2307.947371
Manufacture of industrial and domestic machinery and equipment,Manufacture of tobacco products,28.30833775
Manufacture of industrial and domestic machinery and equipment,Manufacture of textile products,366.8814746
Manufacture of industrial and domestic machinery and equipment,Manufacture of wearing apparel,1266.87274
Manufacture of industrial and domestic machinery and equipment,Manufacture of leather products,72.81286979
Manufacture of industrial and domestic machinery and equipment,Manufacture of footwear,337.477411
Manufacture of industrial and domestic machinery and equipment,Sawmilling and planing of wood,367.5872322
Manufacture of industrial and domestic machinery and equipment,Manufacture of wood products,1767.207361
Manufacture of industrial and domestic machinery and equipment,Manufacture of pulp and paper,2642.138569
Manufacture of industrial and domestic machinery and equipment,Manufacture of containers of paper,1247.486856
Manufacture of industrial and domestic machinery and equipment,Manufacture of other paper products,1250.129861
Manufacture of industrial and domestic machinery and equipment,Printing,664.8867628
Manufacture of industrial and domestic machinery and equipment,Manufacture of refined petroleum products,3752.334197
Manufacture of industrial and domestic machinery and equipment,Manufacture of basic chemicals,997.4705201
Manufacture of industrial and domestic machinery and equipment,Manufacture of paints,392.6239189
Manufacture of industrial and domestic machinery and equipment,Manufacture of pharmaceutical products,1061.650998
Manufacture of industrial and domestic machinery and equipment,Manufacture of soap detergents and toilet products,2108.945612
Manufacture of industrial and domestic machinery and equipment,Manufacture of other chemical products,773.0484847
Manufacture of industrial and domestic machinery and equipment,Manufacture of rubber products,473.0393708
Manufacture of industrial and domestic machinery and equipment,Manufacture of plastic products,1755.932884
Manufacture of industrial and domestic machinery and equipment,Manufacture of glass and glass products,877.9234642
Manufacture of industrial and domestic machinery and equipment,Manufacture of cement,2877.750629
Manufacture of industrial and domestic machinery and equipment,Manufacture of concrete and concrete products,2535.34194
Manufacture of industrial and domestic machinery and equipment,Manufacture of basic iron and steel,7051.633736
Manufacture of industrial and domestic machinery and equipment,Manufacture of other basic metals,2663.673846
Manufacture of industrial and domestic machinery and equipment,Manufacture of fabricated metal products,5430.920336
Manufacture of industrial and domestic machinery and equipment,Manufacture of industrial and domestic machinery and equipment,5067.533462
Manufacture of industrial and domestic machinery and equipment,Manufacture of electric and electronic machinery and equipment,3383.374303
Manufacture of industrial and domestic machinery and equipment,Manufacture of transport equipment,1717.764739
Manufacture of industrial and domestic machinery and equipment,Manufacture of furniture,1527.618156
Manufacture of industrial and domestic machinery and equipment,Repair and installation of machinery and other manufacturing,3036.709638
Manufacture of industrial and domestic machinery and equipment,Electric power generation,8507.821416
Manufacture of industrial and domestic machinery and equipment,Transmission of electric power,1061.812789
Manufacture of industrial and domestic machinery and equipment,Distribution of electric power,3812.241916
Manufacture of industrial and domestic machinery and equipment,Gas and steam manufacture and supply,1279.015427
Manufacture of industrial and domestic machinery and equipment,Water collection treatment and supply,314.3877442
Manufacture of industrial and domestic machinery and equipment,Waste collection and recycling activities,7218.716531
Manufacture of industrial and domestic machinery and equipment,Construction of residential buildings,4516.870032
Manufacture of industrial and domestic machinery and equipment,Construction of non-residential buildings,3310.392619
Manufacture of industrial and domestic machinery and equipment,Civil engineering,25887.20939
Manufacture of industrial and domestic machinery and equipment,Specialized construction activities,21849.65304
Manufacture of industrial and domestic machinery and equipment,Wholesale and retail trade of motor vehicles,21640.60695
Manufacture of industrial and domestic machinery and equipment,Wholesale trade,80609.06523
Manufacture of industrial and domestic machinery and equipment,Retail trade,11713.35902
Manufacture of industrial and domestic machinery and equipment,Accommodation,749.5209699
Manufacture of industrial and domestic machinery and equipment,Food and beverage service activities,3163.692592
Manufacture of industrial and domestic machinery and equipment,Transport via railways,1357.971887
Manufacture of industrial and domestic machinery and equipment,Other passenger land transport,10697.24804
Manufacture of industrial and domestic machinery and equipment,Freight transport by road,11512.21509
Manufacture of industrial and domestic machinery and equipment,Transport via pipeline,3300.695661
Manufacture of industrial and domestic machinery and equipment,Water transport,1076.039771
Manufacture of industrial and domestic machinery and equipment,Air transport,22562.05411
Manufacture of industrial and domestic machinery and equipment,Warehousing,198.9177617
Manufacture of industrial and domestic machinery and equipment,Support activities for land transport,279.2419667
Manufacture of industrial and domestic machinery and equipment,Other support activities for transport,5096.712217
Manufacture of industrial and domestic machinery and equipment,Postal and courier activities,146.8885233
Manufacture of industrial and domestic machinery and equipment,Wireless telecommunications,5182.285817
Manufacture of industrial and domestic machinery and equipment,Wired telecommunications,4792.278229
Manufacture of industrial and domestic machinery and equipment,Other telecommunications activities,6354.944682
Manufacture of industrial and domestic machinery and equipment,Information services activities,15805.60321
Manufacture of industrial and domestic machinery and equipment,Publishing activities,1503.026008
Manufacture of industrial and domestic machinery and equipment,Banking monetary intermediation,1224.679628
Manufacture of industrial and domestic machinery and equipment,Insurance activities,417.8544563
Manufacture of industrial and domestic machinery and equipment,Auxiliary financial activities,1180.658777
Manufacture of industrial and domestic machinery and equipment,Real estate activities,7552.468113
Manufacture of industrial and domestic machinery and equipment,Housing services,0
Manufacture of industrial and domestic machinery and equipment,Legal and accounting services,539.3221391
Manufacture of industrial and domestic machinery and equipment,Architectural and engineering services,3895.342859
Manufacture of industrial and domestic machinery and equipment,Other professional activities,3621.742247
Manufacture of industrial and domestic machinery and equipment,Rental and leasing activities,29160.84734
Manufacture of industrial and domestic machinery and equipment,Business support activities,12220.36025
Manufacture of industrial and domestic machinery and equipment,Public administration,6949.284741
Manufacture of industrial and domestic machinery and equipment,Public education,6654.72649
Manufacture of industrial and domestic machinery and equipment,Private education,2894.987719
Manufacture of industrial and domestic machinery and equipment,Public health activities,7523.794742
Manufacture of industrial and domestic machinery and equipment,Private health activities,7539.48006
Manufacture of industrial and domestic machinery and equipment,Activities of organizations,1883.714264
Manufacture of industrial and domestic machinery and equipment,Artistic and entertainment activities,4466.948313
Manufacture of industrial and domestic machinery and equipment,Other personal services,1965.898361
Manufacture of electric and electronic machinery and equipment,Cultivation of annual crops,448.8047857
Manufacture of electric and electronic machinery and equipment,Cultivation of vegetables,108.7283565
Manufacture of electric and electronic machinery and equipment,Cultivation of grapes,1287.00486
Manufacture of electric and electronic machinery and equipment,Cultivation of other fruits,1777.437567
Manufacture of electric and electronic machinery and equipment,Cattle bredding,348.1266359
Manufacture of electric and electronic machinery and equipment,Pigs breeding,41.92430361
Manufacture of electric and electronic machinery and equipment,Chicken breeding,658.1897647
Manufacture of electric and electronic machinery and equipment,Breeding of other animals,22.91282371
Manufacture of electric and electronic machinery and equipment,Farming services,773.5987069
Manufacture of electric and electronic machinery and equipment,Forestry,298.7582214
Manufacture of electric and electronic machinery and equipment,Aquaculture,156.9768803
Manufacture of electric and electronic machinery and equipment,Extractive fishing,2240.840324
Manufacture of electric and electronic machinery and equipment,Coal mining,18.89664858
Manufacture of electric and electronic machinery and equipment,Crude oil and gas mining,58.31513996
Manufacture of electric and electronic machinery and equipment,Copper mining,10188.02205
Manufacture of electric and electronic machinery and equipment,Iron mining,709.2091569
Manufacture of electric and electronic machinery and equipment,Mining of other metals,1358.943767
Manufacture of electric and electronic machinery and equipment,Other mining activities and mining services,514.1089829
Manufacture of electric and electronic machinery and equipment,Processing and preserving of meat,291.8805957
Manufacture of electric and electronic machinery and equipment,Processing and preserving of fishmeal and fish oil,72.33669206
Manufacture of electric and electronic machinery and equipment,Processing and preserving of fish,433.9908874
Manufacture of electric and electronic machinery and equipment,Processing and preserving of fruits and vegetables,277.7987553
Manufacture of electric and electronic machinery and equipment,Manufacture of vegetable and animal oils,68.2015982
Manufacture of electric and electronic machinery and equipment,Manufacture of dairy products,352.0709427
Manufacture of electric and electronic machinery and equipment,Manufacture of grain mill products,143.1359382
Manufacture of electric and electronic machinery and equipment,Manufacture of prepared animal feeds,387.2638738
Manufacture of electric and electronic machinery and equipment,Manufacture of bakery products,309.6002355
Manufacture of electric and electronic machinery and equipment,Manufacture of macaroni noodles and other related products,15.64678324
Manufacture of electric and electronic machinery and equipment,Manufacture of other food products,1608.623062
Manufacture of electric and electronic machinery and equipment,Distilling rectifying and blending of spirits,34.79582388
Manufacture of electric and electronic machinery and equipment,Manufacture of wines,442.7353705
Manufacture of electric and electronic machinery and equipment,Manufacture of beers,334.7187787
Manufacture of electric and electronic machinery and equipment,Manufacture of soft drinks,369.1449651
Manufacture of electric and electronic machinery and equipment,Manufacture of tobacco products,19.37680913
Manufacture of electric and electronic machinery and equipment,Manufacture of textile products,288.0559559
Manufacture of electric and electronic machinery and equipment,Manufacture of wearing apparel,458.3040774
Manufacture of electric and electronic machinery and equipment,Manufacture of leather products,188.9536246
Manufacture of electric and electronic machinery and equipment,Manufacture of footwear,81.57345201
Manufacture of electric and electronic machinery and equipment,Sawmilling and planing of wood,257.7966045
Manufacture of electric and electronic machinery and equipment,Manufacture of wood products,866.2264954
Manufacture of electric and electronic machinery and equipment,Manufacture of pulp and paper,768.2573089
Manufacture of electric and electronic machinery and equipment,Manufacture of containers of paper,500.4791603
Manufacture of electric and electronic machinery and equipment,Manufacture of other paper products,645.3019787
Manufacture of electric and electronic machinery and equipment,Printing,210.6686115
Manufacture of electric and electronic machinery and equipment,Manufacture of refined petroleum products,77.13749652
Manufacture of electric and electronic machinery and equipment,Manufacture of basic chemicals,2224.929346
Manufacture of electric and electronic machinery and equipment,Manufacture of paints,62.41108692
Manufacture of electric and electronic machinery and equipment,Manufacture of pharmaceutical products,464.5053006
Manufacture of electric and electronic machinery and equipment,Manufacture of soap detergents and toilet products,277.5007324
Manufacture of electric and electronic machinery and equipment,Manufacture of other chemical products,341.7799555
Manufacture of electric and electronic machinery and equipment,Manufacture of rubber products,1352.532373
Manufacture of electric and electronic machinery and equipment,Manufacture of plastic products,611.63669
Manufacture of electric and electronic machinery and equipment,Manufacture of glass and glass products,405.0275826
Manufacture of electric and electronic machinery and equipment,Manufacture of cement,403.2018137
Manufacture of electric and electronic machinery and equipment,Manufacture of concrete and concrete products,389.2145454
Manufacture of electric and electronic machinery and equipment,Manufacture of basic iron and steel,1383.785572
Manufacture of electric and electronic machinery and equipment,Manufacture of other basic metals,4012.312786
Manufacture of electric and electronic machinery and equipment,Manufacture of fabricated metal products,1626.49901
Manufacture of electric and electronic machinery and equipment,Manufacture of industrial and domestic machinery and equipment,1525.106385
Manufacture of electric and electronic machinery and equipment,Manufacture of electric and electronic machinery and equipment,36444.85548
Manufacture of electric and electronic machinery and equipment,Manufacture of transport equipment,11526.70454
Manufacture of electric and electronic machinery and equipment,Manufacture of furniture,334.1338913
Manufacture of electric and electronic machinery and equipment,Repair and installation of machinery and other manufacturing,2570.097554
Manufacture of electric and electronic machinery and equipment,Electric power generation,2347.851086
Manufacture of electric and electronic machinery and equipment,Transmission of electric power,319.9975779
Manufacture of electric and electronic machinery and equipment,Distribution of electric power,997.7606784
Manufacture of electric and electronic machinery and equipment,Gas and steam manufacture and supply,418.0020017
Manufacture of electric and electronic machinery and equipment,Water collection treatment and supply,96.64505836
Manufacture of electric and electronic machinery and equipment,Waste collection and recycling activities,1922.523035
Manufacture of electric and electronic machinery and equipment,Construction of residential buildings,18742.69742
Manufacture of electric and electronic machinery and equipment,Construction of non-residential buildings,16662.51536
Manufacture of electric and electronic machinery and equipment,Civil engineering,8242.71581
Manufacture of electric and electronic machinery and equipment,Specialized construction activities,24141.22777
Manufacture of electric and electronic machinery and equipment,Wholesale and retail trade of motor vehicles,3246.253408
Manufacture of electric and electronic machinery and equipment,Wholesale trade,12399.63896
Manufacture of electric and electronic machinery and equipment,Retail trade,4193.721424
Manufacture of electric and electronic machinery and equipment,Accommodation,1230.820941
Manufacture of electric and electronic machinery and equipment,Food and beverage service activities,1067.013872
Manufacture of electric and electronic machinery and equipment,Transport via railways,307.0955451
Manufacture of electric and electronic machinery and equipment,Other passenger land transport,1848.797785
Manufacture of electric and electronic machinery and equipment,Freight transport by road,3360.500185
Manufacture of electric and electronic machinery and equipment,Transport via pipeline,263.9062741
Manufacture of electric and electronic machinery and equipment,Water transport,291.7312715
Manufacture of electric and electronic machinery and equipment,Air transport,529.948269
Manufacture of electric and electronic machinery and equipment,Warehousing,48.82305408
Manufacture of electric and electronic machinery and equipment,Support activities for land transport,135.6889855
Manufacture of electric and electronic machinery and equipment,Other support activities for transport,1244.617985
Manufacture of electric and electronic machinery and equipment,Postal and courier activities,62.15803134
Manufacture of electric and electronic machinery and equipment,Wireless telecommunications,2694.043227
Manufacture of electric and electronic machinery and equipment,Wired telecommunications,1881.053445
Manufacture of electric and electronic machinery and equipment,Other telecommunications activities,2621.366727
Manufacture of electric and electronic machinery and equipment,Information services activities,587.3657519
Manufacture of electric and electronic machinery and equipment,Publishing activities,5849.513351
Manufacture of electric and electronic machinery and equipment,Banking monetary intermediation,660.2751958
Manufacture of electric and electronic machinery and equipment,Insurance activities,226.5253251
Manufacture of electric and electronic machinery and equipment,Auxiliary financial activities,5556.414392
Manufacture of electric and electronic machinery and equipment,Real estate activities,1932.419551
Manufacture of electric and electronic machinery and equipment,Housing services,0
Manufacture of electric and electronic machinery and equipment,Legal and accounting services,225.3098082
Manufacture of electric and electronic machinery and equipment,Architectural and engineering services,1854.867753
Manufacture of electric and electronic machinery and equipment,Other professional activities,861.0752147
Manufacture of electric and electronic machinery and equipment,Rental and leasing activities,2208.416387
Manufacture of electric and electronic machinery and equipment,Business support activities,30942.40221
Manufacture of electric and electronic machinery and equipment,Public administration,16891.5599
Manufacture of electric and electronic machinery and equipment,Public education,4024.008778
Manufacture of electric and electronic machinery and equipment,Private education,2247.843559
Manufacture of electric and electronic machinery and equipment,Public health activities,20132.21849
Manufacture of electric and electronic machinery and equipment,Private health activities,8725.888443
Manufacture of electric and electronic machinery and equipment,Activities of organizations,1050.539058
Manufacture of electric and electronic machinery and equipment,Artistic and entertainment activities,4784.357251
Manufacture of electric and electronic machinery and equipment,Other personal services,11733.0128
Manufacture of transport equipment,Cultivation of annual crops,254.9009863
Manufacture of transport equipment,Cultivation of vegetables,1290.122117
Manufacture of transport equipment,Cultivation of grapes,675.3368757
Manufacture of transport equipment,Cultivation of other fruits,2862.501211
Manufacture of transport equipment,Cattle bredding,31.61958958
Manufacture of transport equipment,Pigs breeding,7.277306834
Manufacture of transport equipment,Chicken breeding,17.81097677
Manufacture of transport equipment,Breeding of other animals,2.560686177
Manufacture of transport equipment,Farming services,7895.516323
Manufacture of transport equipment,Forestry,350.9683166
Manufacture of transport equipment,Aquaculture,187.6566754
Manufacture of transport equipment,Extractive fishing,274.3122053
Manufacture of transport equipment,Coal mining,129.0258196
Manufacture of transport equipment,Crude oil and gas mining,814.2133572
Manufacture of transport equipment,Copper mining,17183.13361
Manufacture of transport equipment,Iron mining,1666.403162
Manufacture of transport equipment,Mining of other metals,1559.259181
Manufacture of transport equipment,Other mining activities and mining services,1162.383743
Manufacture of transport equipment,Processing and preserving of meat,79.17670847
Manufacture of transport equipment,Processing and preserving of fishmeal and fish oil,84.52444042
Manufacture of transport equipment,Processing and preserving of fish,421.0638965
Manufacture of transport equipment,Processing and preserving of fruits and vegetables,257.0871266
Manufacture of transport equipment,Manufacture of vegetable and animal oils,10.60923403
Manufacture of transport equipment,Manufacture of dairy products,46.25733885
Manufacture of transport equipment,Manufacture of grain mill products,220.1014827
Manufacture of transport equipment,Manufacture of prepared animal feeds,264.2443719
Manufacture of transport equipment,Manufacture of bakery products,307.6643432
Manufacture of transport equipment,Manufacture of macaroni noodles and other related products,3.501452607
Manufacture of transport equipment,Manufacture of other food products,432.4533575
Manufacture of transport equipment,Distilling rectifying and blending of spirits,8.815424388
Manufacture of transport equipment,Manufacture of wines,331.7133327
Manufacture of transport equipment,Manufacture of beers,230.7060496
Manufacture of transport equipment,Manufacture of soft drinks,324.8705059
Manufacture of transport equipment,Manufacture of tobacco products,2.013123594
Manufacture of transport equipment,Manufacture of textile products,77.55348855
Manufacture of transport equipment,Manufacture of wearing apparel,487.5291856
Manufacture of transport equipment,Manufacture of leather products,26.83447287
Manufacture of transport equipment,Manufacture of footwear,59.79250302
Manufacture of transport equipment,Sawmilling and planing of wood,70.09339585
Manufacture of transport equipment,Manufacture of wood products,982.8359344
Manufacture of transport equipment,Manufacture of pulp and paper,1435.391435
Manufacture of transport equipment,Manufacture of containers of paper,680.0548987
Manufacture of transport equipment,Manufacture of other paper products,565.3407705
Manufacture of transport equipment,Printing,343.8682496
Manufacture of transport equipment,Manufacture of refined petroleum products,36.5270175
Manufacture of transport equipment,Manufacture of basic chemicals,444.4573788
Manufacture of transport equipment,Manufacture of paints,59.36099032
Manufacture of transport equipment,Manufacture of pharmaceutical products,342.6816616
Manufacture of transport equipment,Manufacture of soap detergents and toilet products,348.947998
Manufacture of transport equipment,Manufacture of other chemical products,283.9646194
Manufacture of transport equipment,Manufacture of rubber products,197.8835289
Manufacture of transport equipment,Manufacture of plastic products,570.4799743
Manufacture of transport equipment,Manufacture of glass and glass products,425.3500923
Manufacture of transport equipment,Manufacture of cement,496.3230535
Manufacture of transport equipment,Manufacture of concrete and concrete products,333.6248264
Manufacture of transport equipment,Manufacture of basic iron and steel,2334.434002
Manufacture of transport equipment,Manufacture of other basic metals,1238.36931
Manufacture of transport equipment,Manufacture of fabricated metal products,462.0486883
Manufacture of transport equipment,Manufacture of industrial and domestic machinery and equipment,1504.448268
Manufacture of transport equipment,Manufacture of electric and electronic machinery and equipment,7141.483148
Manufacture of transport equipment,Manufacture of transport equipment,27585.98031
Manufacture of transport equipment,Manufacture of furniture,136.9107224
Manufacture of transport equipment,Repair and installation of machinery and other manufacturing,708.991097
Manufacture of transport equipment,Electric power generation,6391.850564
Manufacture of transport equipment,Transmission of electric power,660.29668
Manufacture of transport equipment,Distribution of electric power,2448.1277
Manufacture of transport equipment,Gas and steam manufacture and supply,364.8113165
Manufacture of transport equipment,Water collection treatment and supply,188.4374863
Manufacture of transport equipment,Waste collection and recycling activities,4570.347394
Manufacture of transport equipment,Construction of residential buildings,1027.732469
Manufacture of transport equipment,Construction of non-residential buildings,965.1389238
Manufacture of transport equipment,Civil engineering,3530.658367
Manufacture of transport equipment,Specialized construction activities,6706.477413
Manufacture of transport equipment,Wholesale and retail trade of motor vehicles,53686.54108
Manufacture of transport equipment,Wholesale trade,6182.597851
Manufacture of transport equipment,Retail trade,2720.042471
Manufacture of transport equipment,Accommodation,344.1361502
Manufacture of transport equipment,Food and beverage service activities,1129.901072
Manufacture of transport equipment,Transport via railways,1416.257054
Manufacture of transport equipment,Other passenger land transport,29764.16231
Manufacture of transport equipment,Freight transport by road,6633.316433
Manufacture of transport equipment,Transport via pipeline,378.4437611
Manufacture of transport equipment,Water transport,641.6471003
Manufacture of transport equipment,Air transport,11246.71542
Manufacture of transport equipment,Warehousing,2013.374091
Manufacture of transport equipment,Support activities for land transport,165.6913283
Manufacture of transport equipment,Other support activities for transport,7129.875694
Manufacture of transport equipment,Postal and courier activities,88.46033981
Manufacture of transport equipment,Wireless telecommunications,2006.301599
Manufacture of transport equipment,Wired telecommunications,2751.369575
Manufacture of transport equipment,Other telecommunications activities,3570.136231
Manufacture of transport equipment,Information services activities,828.9426316
Manufacture of transport equipment,Publishing activities,532.1749006
Manufacture of transport equipment,Banking monetary intermediation,846.259265
Manufacture of transport equipment,Insurance activities,186.620304
Manufacture of transport equipment,Auxiliary financial activities,1207.503132
Manufacture of transport equipment,Real estate activities,1261.261788
Manufacture of transport equipment,Housing services,0
Manufacture of transport equipment,Legal and accounting services,115.6737161
Manufacture of transport equipment,Architectural and engineering services,2259.907167
Manufacture of transport equipment,Other professional activities,1002.129279
Manufacture of transport equipment,Rental and leasing activities,10890.13967
Manufacture of transport equipment,Business support activities,7270.082391
Manufacture of transport equipment,Public administration,9340.652801
Manufacture of transport equipment,Public education,796.1761611
Manufacture of transport equipment,Private education,795.9014876
Manufacture of transport equipment,Public health activities,2359.278453
Manufacture of transport equipment,Private health activities,3126.884196
Manufacture of transport equipment,Activities of organizations,198.6690421
Manufacture of transport equipment,Artistic and entertainment activities,551.7443536
Manufacture of transport equipment,Other personal services,685.8962826
Manufacture of furniture,Cultivation of annual crops,165.3242509
Manufacture of furniture,Cultivation of vegetables,95.58649784
Manufacture of furniture,Cultivation of grapes,102.0849718
Manufacture of furniture,Cultivation of other fruits,265.3242858
Manufacture of furniture,Cattle bredding,107.955775
Manufacture of furniture,Pigs breeding,21.86660693
Manufacture of furniture,Chicken breeding,112.1911924
Manufacture of furniture,Breeding of other animals,9.713076572
Manufacture of furniture,Farming services,62.16955475
Manufacture of furniture,Forestry,59.55327649
Manufacture of furniture,Aquaculture,149.6745293
Manufacture of furniture,Extractive fishing,222.1028265
Manufacture of furniture,Coal mining,6.307934341
Manufacture of furniture,Crude oil and gas mining,4.709233945
Manufacture of furniture,Copper mining,1147.164881
Manufacture of furniture,Iron mining,30.76924978
Manufacture of furniture,Mining of other metals,53.3816665
Manufacture of furniture,Other mining activities and mining services,35.19843857
Manufacture of furniture,Processing and preserving of meat,182.5546476
Manufacture of furniture,Processing and preserving of fishmeal and fish oil,16.61485582
Manufacture of furniture,Processing and preserving of fish,113.7537707
Manufacture of furniture,Processing and preserving of fruits and vegetables,116.7694537
Manufacture of furniture,Manufacture of vegetable and animal oils,36.80645246
Manufacture of furniture,Manufacture of dairy products,244.1537058
Manufacture of furniture,Manufacture of grain mill products,116.0586845
Manufacture of furniture,Manufacture of prepared animal feeds,203.9191393
Manufacture of furniture,Manufacture of bakery products,124.0025813
Manufacture of furniture,Manufacture of macaroni noodles and other related products,20.19577236
Manufacture of furniture,Manufacture of other food products,312.4069886
Manufacture of furniture,Distilling rectifying and blending of spirits,13.23575198
Manufacture of furniture,Manufacture of wines,322.7610693
Manufacture of furniture,Manufacture of beers,151.5530377
Manufacture of furniture,Manufacture of soft drinks,446.2647295
Manufacture of furniture,Manufacture of tobacco products,36.77486191
Manufacture of furniture,Manufacture of textile products,798.8305776
Manufacture of furniture,Manufacture of wearing apparel,1486.925132
Manufacture of furniture,Manufacture of leather products,455.5319871
Manufacture of furniture,Manufacture of footwear,213.4977679
Manufacture of furniture,Sawmilling and planing of wood,280.3243363
Manufacture of furniture,Manufacture of wood products,509.4068255
Manufacture of furniture,Manufacture of pulp and paper,126.8494257
Manufacture of furniture,Manufacture of containers of paper,83.72434647
Manufacture of furniture,Manufacture of other paper products,435.1526937
Manufacture of furniture,Printing,48.74403248
Manufacture of furniture,Manufacture of refined petroleum products,88.83972146
Manufacture of furniture,Manufacture of basic chemicals,137.6846431
Manufacture of furniture,Manufacture of paints,68.25176055
Manufacture of furniture,Manufacture of pharmaceutical products,213.1736658
Manufacture of furniture,Manufacture of soap detergents and toilet products,174.0561254
Manufacture of furniture,Manufacture of other chemical products,61.6931538
Manufacture of furniture,Manufacture of rubber products,330.6737943
Manufacture of furniture,Manufacture of plastic products,327.9377015
Manufacture of furniture,Manufacture of glass and glass products,48.85221741
Manufacture of furniture,Manufacture of cement,53.56312889
Manufacture of furniture,Manufacture of concrete and concrete products,185.4780145
Manufacture of furniture,Manufacture of basic iron and steel,144.9641846
Manufacture of furniture,Manufacture of other basic metals,81.50325292
Manufacture of furniture,Manufacture of fabricated metal products,693.0985861
Manufacture of furniture,Manufacture of industrial and domestic machinery and equipment,169.7251646
Manufacture of furniture,Manufacture of electric and electronic machinery and equipment,100.8173825
Manufacture of furniture,Manufacture of transport equipment,200.910458
Manufacture of furniture,Manufacture of furniture,10147.96646
Manufacture of furniture,Repair and installation of machinery and other manufacturing,229.6767567
Manufacture of furniture,Electric power generation,1064.8812
Manufacture of furniture,Transmission of electric power,6.959160989
Manufacture of furniture,Distribution of electric power,43.95809928
Manufacture of furniture,Gas and steam manufacture and supply,61.6362069
Manufacture of furniture,Water collection treatment and supply,11.39523462
Manufacture of furniture,Waste collection and recycling activities,53.25457715
Manufacture of furniture,Construction of residential buildings,56178.50662
Manufacture of furniture,Construction of non-residential buildings,3868.946774
Manufacture of furniture,Civil engineering,5947.143538
Manufacture of furniture,Specialized construction activities,4612.995336
Manufacture of furniture,Wholesale and retail trade of motor vehicles,400.0468454
Manufacture of furniture,Wholesale trade,1243.020314
Manufacture of furniture,Retail trade,2004.808218
Manufacture of furniture,Accommodation,137.0864397
Manufacture of furniture,Food and beverage service activities,306.4321224
Manufacture of furniture,Transport via railways,14.342196
Manufacture of furniture,Other passenger land transport,172.5103842
Manufacture of furniture,Freight transport by road,259.7977013
Manufacture of furniture,Transport via pipeline,5.465752246
Manufacture of furniture,Water transport,21.21802635
Manufacture of furniture,Air transport,144.7330605
Manufacture of furniture,Warehousing,10.47349292
Manufacture of furniture,Support activities for land transport,20.89525823
Manufacture of furniture,Other support activities for transport,144.831505
Manufacture of furniture,Postal and courier activities,2.597343252
Manufacture of furniture,Wireless telecommunications,409.5617027
Manufacture of furniture,Wired telecommunications,206.2324982
Manufacture of furniture,Other telecommunications activities,196.2123744
Manufacture of furniture,Information services activities,565.4852042
Manufacture of furniture,Publishing activities,106.0105538
Manufacture of furniture,Banking monetary intermediation,515.5691298
Manufacture of furniture,Insurance activities,152.8346664
Manufacture of furniture,Auxiliary financial activities,132.097086
Manufacture of furniture,Real estate activities,117.5829152
Manufacture of furniture,Housing services,0
Manufacture of furniture,Legal and accounting services,58.37122006
Manufacture of furniture,Architectural and engineering services,235.0125952
Manufacture of furniture,Other professional activities,473.9936481
Manufacture of furniture,Rental and leasing activities,170.0741367
Manufacture of furniture,Business support activities,518.8431466
Manufacture of furniture,Public administration,357.8348034
Manufacture of furniture,Public education,70.99175535
Manufacture of furniture,Private education,99.01909598
Manufacture of furniture,Public health activities,520.7279522
Manufacture of furniture,Private health activities,702.8443655
Manufacture of furniture,Activities of organizations,984.8982067
Manufacture of furniture,Artistic and entertainment activities,271.28052
Manufacture of furniture,Other personal services,72.659279
Repair and installation of machinery and other manufacturing,Cultivation of annual crops,216.5632061
Repair and installation of machinery and other manufacturing,Cultivation of vegetables,58.83978992
Repair and installation of machinery and other manufacturing,Cultivation of grapes,126.1125356
Repair and installation of machinery and other manufacturing,Cultivation of other fruits,299.9755439
Repair and installation of machinery and other manufacturing,Cattle bredding,136.1716026
Repair and installation of machinery and other manufacturing,Pigs breeding,23.29349979
Repair and installation of machinery and other manufacturing,Chicken breeding,33.47696746
Repair and installation of machinery and other manufacturing,Breeding of other animals,13.95409911
Repair and installation of machinery and other manufacturing,Farming services,79.34611914
Repair and installation of machinery and other manufacturing,Forestry,997.848259
Repair and installation of machinery and other manufacturing,Aquaculture,308.850562
Repair and installation of machinery and other manufacturing,Extractive fishing,1463.545446
Repair and installation of machinery and other manufacturing,Coal mining,113.8640944
Repair and installation of machinery and other manufacturing,Crude oil and gas mining,612.4462663
Repair and installation of machinery and other manufacturing,Copper mining,87000.0412
Repair and installation of machinery and other manufacturing,Iron mining,8923.115271
Repair and installation of machinery and other manufacturing,Mining of other metals,8247.611056
Repair and installation of machinery and other manufacturing,Other mining activities and mining services,6073.356559
Repair and installation of machinery and other manufacturing,Processing and preserving of meat,448.3520558
Repair and installation of machinery and other manufacturing,Processing and preserving of fishmeal and fish oil,458.6696936
Repair and installation of machinery and other manufacturing,Processing and preserving of fish,2259.912456
Repair and installation of machinery and other manufacturing,Processing and preserving of fruits and vegetables,1047.875346
Repair and installation of machinery and other manufacturing,Manufacture of vegetable and animal oils,40.92897556
Repair and installation of machinery and other manufacturing,Manufacture of dairy products,244.6296468
Repair and installation of machinery and other manufacturing,Manufacture of grain mill products,1091.849615
Repair and installation of machinery and other manufacturing,Manufacture of prepared animal feeds,1465.065644
Repair and installation of machinery and other manufacturing,Manufacture of bakery products,1614.035893
Repair and installation of machinery and other manufacturing,Manufacture of macaroni noodles and other related products,26.3777445
Repair and installation of machinery and other manufacturing,Manufacture of other food products,2234.300575
Repair and installation of machinery and other manufacturing,Distilling rectifying and blending of spirits,10.31053448
Repair and installation of machinery and other manufacturing,Manufacture of wines,1742.8778
Repair and installation of machinery and other manufacturing,Manufacture of beers,513.0312879
Repair and installation of machinery and other manufacturing,Manufacture of soft drinks,1677.21945
Repair and installation of machinery and other manufacturing,Manufacture of tobacco products,30.03128209
Repair and installation of machinery and other manufacturing,Manufacture of textile products,389.804518
Repair and installation of machinery and other manufacturing,Manufacture of wearing apparel,2558.956536
Repair and installation of machinery and other manufacturing,Manufacture of leather products,140.4089625
Repair and installation of machinery and other manufacturing,Manufacture of footwear,297.1500231
Repair and installation of machinery and other manufacturing,Sawmilling and planing of wood,58.86148236
Repair and installation of machinery and other manufacturing,Manufacture of wood products,5052.776215
Repair and installation of machinery and other manufacturing,Manufacture of pulp and paper,7424.325307
Repair and installation of machinery and other manufacturing,Manufacture of containers of paper,3682.926714
Repair and installation of machinery and other manufacturing,Manufacture of other paper products,3047.817324
Repair and installation of machinery and other manufacturing,Printing,1825.651636
Repair and installation of machinery and other manufacturing,Manufacture of refined petroleum products,72.09857558
Repair and installation of machinery and other manufacturing,Manufacture of basic chemicals,2384.323284
Repair and installation of machinery and other manufacturing,Manufacture of paints,60.41921409
Repair and installation of machinery and other manufacturing,Manufacture of pharmaceutical products,1756.299138
Repair and installation of machinery and other manufacturing,Manufacture of soap detergents and toilet products,1844.738203
Repair and installation of machinery and other manufacturing,Manufacture of other chemical products,1419.976603
Repair and installation of machinery and other manufacturing,Manufacture of rubber products,962.1983339
Repair and installation of machinery and other manufacturing,Manufacture of plastic products,3156.18544
Repair and installation of machinery and other manufacturing,Manufacture of glass and glass products,2141.042914
Repair and installation of machinery and other manufacturing,Manufacture of cement,2648.59476
Repair and installation of machinery and other manufacturing,Manufacture of concrete and concrete products,500.7394532
Repair and installation of machinery and other manufacturing,Manufacture of basic iron and steel,11756.40835
Repair and installation of machinery and other manufacturing,Manufacture of other basic metals,6824.996926
Repair and installation of machinery and other manufacturing,Manufacture of fabricated metal products,1580.776215
Repair and installation of machinery and other manufacturing,Manufacture of industrial and domestic machinery and equipment,5797.713368
Repair and installation of machinery and other manufacturing,Manufacture of electric and electronic machinery and equipment,2352.487274
Repair and installation of machinery and other manufacturing,Manufacture of transport equipment,100.066982
Repair and installation of machinery and other manufacturing,Manufacture of furniture,516.5439155
Repair and installation of machinery and other manufacturing,Repair and installation of machinery and other manufacturing,1142.887065
Repair and installation of machinery and other manufacturing,Electric power generation,27919.82725
Repair and installation of machinery and other manufacturing,Transmission of electric power,3551.329971
Repair and installation of machinery and other manufacturing,Distribution of electric power,13016.92722
Repair and installation of machinery and other manufacturing,Gas and steam manufacture and supply,1796.983588
Repair and installation of machinery and other manufacturing,Water collection treatment and supply,1012.169927
Repair and installation of machinery and other manufacturing,Waste collection and recycling activities,24601.64305
Repair and installation of machinery and other manufacturing,Construction of residential buildings,2831.994749
Repair and installation of machinery and other manufacturing,Construction of non-residential buildings,1741.949217
Repair and installation of machinery and other manufacturing,Civil engineering,10544.73076
Repair and installation of machinery and other manufacturing,Specialized construction activities,3262.094976
Repair and installation of machinery and other manufacturing,Wholesale and retail trade of motor vehicles,3350.247405
Repair and installation of machinery and other manufacturing,Wholesale trade,31560.19283
Repair and installation of machinery and other manufacturing,Retail trade,13067.02444
Repair and installation of machinery and other manufacturing,Accommodation,1642.797524
Repair and installation of machinery and other manufacturing,Food and beverage service activities,6119.593033
Repair and installation of machinery and other manufacturing,Transport via railways,3935.466895
Repair and installation of machinery and other manufacturing,Other passenger land transport,20648.15292
Repair and installation of machinery and other manufacturing,Freight transport by road,35317.29703
Repair and installation of machinery and other manufacturing,Transport via pipeline,1993.440921
Repair and installation of machinery and other manufacturing,Water transport,3446.497855
Repair and installation of machinery and other manufacturing,Air transport,4363.948648
Repair and installation of machinery and other manufacturing,Warehousing,466.0389566
Repair and installation of machinery and other manufacturing,Support activities for land transport,877.3254348
Repair and installation of machinery and other manufacturing,Other support activities for transport,10190.38653
Repair and installation of machinery and other manufacturing,Postal and courier activities,476.460686
Repair and installation of machinery and other manufacturing,Wireless telecommunications,10793.59243
Repair and installation of machinery and other manufacturing,Wired telecommunications,14644.1317
Repair and installation of machinery and other manufacturing,Other telecommunications activities,19206.00109
Repair and installation of machinery and other manufacturing,Information services activities,3334.837023
Repair and installation of machinery and other manufacturing,Publishing activities,1324.768048
Repair and installation of machinery and other manufacturing,Banking monetary intermediation,2826.08764
Repair and installation of machinery and other manufacturing,Insurance activities,858.432104
Repair and installation of machinery and other manufacturing,Auxiliary financial activities,2963.578209
Repair and installation of machinery and other manufacturing,Real estate activities,5819.314938
Repair and installation of machinery and other manufacturing,Housing services,0
Repair and installation of machinery and other manufacturing,Legal and accounting services,592.807798
Repair and installation of machinery and other manufacturing,Architectural and engineering services,12025.9724
Repair and installation of machinery and other manufacturing,Other professional activities,3610.634964
Repair and installation of machinery and other manufacturing,Rental and leasing activities,26989.79846
Repair and installation of machinery and other manufacturing,Business support activities,12309.79099
Repair and installation of machinery and other manufacturing,Public administration,14559.81376
Repair and installation of machinery and other manufacturing,Public education,4088.352676
Repair and installation of machinery and other manufacturing,Private education,4248.843059
Repair and installation of machinery and other manufacturing,Public health activities,9712.564833
Repair and installation of machinery and other manufacturing,Private health activities,16186.45308
Repair and installation of machinery and other manufacturing,Activities of organizations,853.8991522
Repair and installation of machinery and other manufacturing,Artistic and entertainment activities,2341.896937
Repair and installation of machinery and other manufacturing,Other personal services,3626.08172
Electric power generation,Cultivation of annual crops,281.5318422
Electric power generation,Cultivation of vegetables,76.44500503
Electric power generation,Cultivation of grapes,159.9271933
Electric power generation,Cultivation of other fruits,387.5840565
Electric power generation,Cattle bredding,193.5137953
Electric power generation,Pigs breeding,35.60466735
Electric power generation,Chicken breeding,47.7225665
Electric power generation,Breeding of other animals,18.31112034
Electric power generation,Farming services,112.8639474
Electric power generation,Forestry,475.5965351
Electric power generation,Aquaculture,336.6822786
Electric power generation,Extractive fishing,123.4405593
Electric power generation,Coal mining,133.6677507
Electric power generation,Crude oil and gas mining,103.2112156
Electric power generation,Copper mining,1428147.6
Electric power generation,Iron mining,41308.96186
Electric power generation,Mining of other metals,26886.55423
Electric power generation,Other mining activities and mining services,25744.53162
Electric power generation,Processing and preserving of meat,32599.22222
Electric power generation,Processing and preserving of fishmeal and fish oil,4123.762844
Electric power generation,Processing and preserving of fish,34454.88713
Electric power generation,Processing and preserving of fruits and vegetables,12055.88734
Electric power generation,Manufacture of vegetable and animal oils,135.2348691
Electric power generation,Manufacture of dairy products,15268.41592
Electric power generation,Manufacture of grain mill products,4690.816312
Electric power generation,Manufacture of prepared animal feeds,630.2284474
Electric power generation,Manufacture of bakery products,1359.900864
Electric power generation,Manufacture of macaroni noodles and other related products,864.8509948
Electric power generation,Manufacture of other food products,13857.33833
Electric power generation,Distilling rectifying and blending of spirits,138.876629
Electric power generation,Manufacture of wines,578.366888
Electric power generation,Manufacture of beers,708.6864364
Electric power generation,Manufacture of soft drinks,503.9140401
Electric power generation,Manufacture of tobacco products,124.8041158
Electric power generation,Manufacture of textile products,339.138407
Electric power generation,Manufacture of wearing apparel,558.1787709
Electric power generation,Manufacture of leather products,33.51981668
Electric power generation,Manufacture of footwear,44.63754626
Electric power generation,Sawmilling and planing of wood,33324.05606
Electric power generation,Manufacture of wood products,39073.63592
Electric power generation,Manufacture of pulp and paper,293826.095
Electric power generation,Manufacture of containers of paper,4512.895564
Electric power generation,Manufacture of other paper products,17882.79964
Electric power generation,Printing,445.0446389
Electric power generation,Manufacture of refined petroleum products,46657.36874
Electric power generation,Manufacture of basic chemicals,82495.67972
Electric power generation,Manufacture of paints,71.94918799
Electric power generation,Manufacture of pharmaceutical products,364.7433263
Electric power generation,Manufacture of soap detergents and toilet products,377.7000352
Electric power generation,Manufacture of other chemical products,255.1729243
Electric power generation,Manufacture of rubber products,2149.814961
Electric power generation,Manufacture of plastic products,40473.73144
Electric power generation,Manufacture of glass and glass products,12257.56741
Electric power generation,Manufacture of cement,32265.41992
Electric power generation,Manufacture of concrete and concrete products,1345.679989
Electric power generation,Manufacture of basic iron and steel,20586.53201
Electric power generation,Manufacture of other basic metals,2848.288452
Electric power generation,Manufacture of fabricated metal products,35340.51279
Electric power generation,Manufacture of industrial and domestic machinery and equipment,10442.679
Electric power generation,Manufacture of electric and electronic machinery and equipment,536.8328173
Electric power generation,Manufacture of transport equipment,4265.613186
Electric power generation,Manufacture of furniture,5477.699427
Electric power generation,Repair and installation of machinery and other manufacturing,808.9830381
Electric power generation,Electric power generation,187050.6393
Electric power generation,Transmission of electric power,1399.177762
Electric power generation,Distribution of electric power,2148941.478
Electric power generation,Gas and steam manufacture and supply,35420.66379
Electric power generation,Water collection treatment and supply,276.3927985
Electric power generation,Waste collection and recycling activities,1450.587865
Electric power generation,Construction of residential buildings,1305.619031
Electric power generation,Construction of non-residential buildings,757.0959969
Electric power generation,Civil engineering,3893.088358
Electric power generation,Specialized construction activities,1653.931818
Electric power generation,Wholesale and retail trade of motor vehicles,957.9553905
Electric power generation,Wholesale trade,5332.112703
Electric power generation,Retail trade,4883.687703
Electric power generation,Accommodation,809.5288365
Electric power generation,Food and beverage service activities,2978.037393
Electric power generation,Transport via railways,224.9046457
Electric power generation,Other passenger land transport,2331.702715
Electric power generation,Freight transport by road,3494.464944
Electric power generation,Transport via pipeline,111.1849533
Electric power generation,Water transport,301.2005367
Electric power generation,Air transport,794.9038428
Electric power generation,Warehousing,139.5077985
Electric power generation,Support activities for land transport,154.0201853
Electric power generation,Other support activities for transport,1096.888018
Electric power generation,Postal and courier activities,54.97644712
Electric power generation,Wireless telecommunications,1311.116218
Electric power generation,Wired telecommunications,1090.34941
Electric power generation,Other telecommunications activities,1433.863941
Electric power generation,Information services activities,781.3825057
Electric power generation,Publishing activities,385.450545
Electric power generation,Banking monetary intermediation,766.3766532
Electric power generation,Insurance activities,456.5698329
Electric power generation,Auxiliary financial activities,300.7685662
Electric power generation,Real estate activities,1142.449429
Electric power generation,Housing services,0
Electric power generation,Legal and accounting services,512.1072852
Electric power generation,Architectural and engineering services,4018.118336
Electric power generation,Other professional activities,1373.981102
Electric power generation,Rental and leasing activities,2051.585622
Electric power generation,Business support activities,2018.508424
Electric power generation,Public administration,1612.913432
Electric power generation,Public education,1204.329307
Electric power generation,Private education,682.298962
Electric power generation,Public health activities,1477.935996
Electric power generation,Private health activities,1903.578377
Electric power generation,Activities of organizations,293.2507509
Electric power generation,Artistic and entertainment activities,747.7683995
Electric power generation,Other personal services,474.9135883
Transmission of electric power,Cultivation of annual crops,4.793715884
Transmission of electric power,Cultivation of vegetables,3.143793328
Transmission of electric power,Cultivation of grapes,5.468119422
Transmission of electric power,Cultivation of other fruits,16.11903775
Transmission of electric power,Cattle bredding,18.21734473
Transmission of electric power,Pigs breeding,6.03172488
Transmission of electric power,Chicken breeding,11.26689847
Transmission of electric power,Breeding of other animals,0.717940847
Transmission of electric power,Farming services,1.708214995
Transmission of electric power,Forestry,87.2958615
Transmission of electric power,Aquaculture,48.76390059
Transmission of electric power,Extractive fishing,1.133757266
Transmission of electric power,Coal mining,1.416772416
Transmission of electric power,Crude oil and gas mining,7.41640848
Transmission of electric power,Copper mining,109675.5485
Transmission of electric power,Iron mining,85.58792944
Transmission of electric power,Mining of other metals,70.86764725
Transmission of electric power,Other mining activities and mining services,57.1197233
Transmission of electric power,Processing and preserving of meat,12.31123864
Transmission of electric power,Processing and preserving of fishmeal and fish oil,1.450072755
Transmission of electric power,Processing and preserving of fish,10.76561852
Transmission of electric power,Processing and preserving of fruits and vegetables,6.964591973
Transmission of electric power,Manufacture of vegetable and animal oils,4.519251603
Transmission of electric power,Manufacture of dairy products,5.573595615
Transmission of electric power,Manufacture of grain mill products,6.822679199
Transmission of electric power,Manufacture of prepared animal feeds,18.13390014
Transmission of electric power,Manufacture of bakery products,17.01432773
Transmission of electric power,Manufacture of macaroni noodles and other related products,0.711681238
Transmission of electric power,Manufacture of other food products,11.96594223
Transmission of electric power,Distilling rectifying and blending of spirits,8.499945604
Transmission of electric power,Manufacture of wines,26.22561062
Transmission of electric power,Manufacture of beers,5.697984429
Transmission of electric power,Manufacture of soft drinks,10.98478714
Transmission of electric power,Manufacture of tobacco products,7.129227399
Transmission of electric power,Manufacture of textile products,10.61417308
Transmission of electric power,Manufacture of wearing apparel,10.56953358
Transmission of electric power,Manufacture of leather products,0.699949058
Transmission of electric power,Manufacture of footwear,1.814452235
Transmission of electric power,Sawmilling and planing of wood,11.24919819
Transmission of electric power,Manufacture of wood products,14.29213941
Transmission of electric power,Manufacture of pulp and paper,25.62668515
Transmission of electric power,Manufacture of containers of paper,4.932817636
Transmission of electric power,Manufacture of other paper products,8.5718624
Transmission of electric power,Printing,18.67750856
Transmission of electric power,Manufacture of refined petroleum products,8.214542464
Transmission of electric power,Manufacture of basic chemicals,48.5633588
Transmission of electric power,Manufacture of paints,1.501548273
Transmission of electric power,Manufacture of pharmaceutical products,13.77566068
Transmission of electric power,Manufacture of soap detergents and toilet products,3.762813937
Transmission of electric power,Manufacture of other chemical products,5.013452099
Transmission of electric power,Manufacture of rubber products,1.703863562
Transmission of electric power,Manufacture of plastic products,16.34737999
Transmission of electric power,Manufacture of glass and glass products,5.255770057
Transmission of electric power,Manufacture of cement,10.07786363
Transmission of electric power,Manufacture of concrete and concrete products,30.07494141
Transmission of electric power,Manufacture of basic iron and steel,15.83682885
Transmission of electric power,Manufacture of other basic metals,6.119572737
Transmission of electric power,Manufacture of fabricated metal products,31.09472429
Transmission of electric power,Manufacture of industrial and domestic machinery and equipment,18.72104066
Transmission of electric power,Manufacture of electric and electronic machinery and equipment,8.948315097
Transmission of electric power,Manufacture of transport equipment,2.678346862
Transmission of electric power,Manufacture of furniture,10.86704953
Transmission of electric power,Repair and installation of machinery and other manufacturing,20.51545666
Transmission of electric power,Electric power generation,187194.6341
Transmission of electric power,Transmission of electric power,1632.112157
Transmission of electric power,Distribution of electric power,124403.7786
Transmission of electric power,Gas and steam manufacture and supply,541.8286289
Transmission of electric power,Water collection treatment and supply,67.58454403
Transmission of electric power,Waste collection and recycling activities,27.668521
Transmission of electric power,Construction of residential buildings,129.2359652
Transmission of electric power,Construction of non-residential buildings,67.45737702
Transmission of electric power,Civil engineering,483.3782056
Transmission of electric power,Specialized construction activities,112.4644385
Transmission of electric power,Wholesale and retail trade of motor vehicles,36.56646049
Transmission of electric power,Wholesale trade,277.1653351
Transmission of electric power,Retail trade,259.7307183
Transmission of electric power,Accommodation,26.83956035
Transmission of electric power,Food and beverage service activities,73.77820723
Transmission of electric power,Transport via railways,7.478260247
Transmission of electric power,Other passenger land transport,74.16540688
Transmission of electric power,Freight transport by road,260.3486684
Transmission of electric power,Transport via pipeline,3.502598051
Transmission of electric power,Water transport,21.86694135
Transmission of electric power,Air transport,43.66141271
Transmission of electric power,Warehousing,16.14948046
Transmission of electric power,Support activities for land transport,24.29846301
Transmission of electric power,Other support activities for transport,88.04159931
Transmission of electric power,Postal and courier activities,3.71828044
Transmission of electric power,Wireless telecommunications,35.1811839
Transmission of electric power,Wired telecommunications,49.49105321
Transmission of electric power,Other telecommunications activities,53.42909447
Transmission of electric power,Information services activities,47.78314443
Transmission of electric power,Publishing activities,30.56234684
Transmission of electric power,Banking monetary intermediation,39.42654203
Transmission of electric power,Insurance activities,15.77622133
Transmission of electric power,Auxiliary financial activities,9.690326159
Transmission of electric power,Real estate activities,91.42802239
Transmission of electric power,Housing services,0
Transmission of electric power,Legal and accounting services,46.12500634
Transmission of electric power,Architectural and engineering services,430.1416662
Transmission of electric power,Other professional activities,101.3346874
Transmission of electric power,Rental and leasing activities,106.2375609
Transmission of electric power,Business support activities,100.7491662
Transmission of electric power,Public administration,246.073195
Transmission of electric power,Public education,68.35899675
Transmission of electric power,Private education,37.38785039
Transmission of electric power,Public health activities,58.54296549
Transmission of electric power,Private health activities,102.8050395
Transmission of electric power,Activities of organizations,8.237651199
Transmission of electric power,Artistic and entertainment activities,58.95780344
Transmission of electric power,Other personal services,10.59907429
Distribution of electric power,Cultivation of annual crops,5305.35208
Distribution of electric power,Cultivation of vegetables,3658.742351
Distribution of electric power,Cultivation of grapes,6272.841819
Distribution of electric power,Cultivation of other fruits,19367.4711
Distribution of electric power,Cattle bredding,23319.59115
Distribution of electric power,Pigs breeding,7763.328261
Distribution of electric power,Chicken breeding,14519.50849
Distribution of electric power,Breeding of other animals,890.6831892
Distribution of electric power,Farming services,70.84243319
Distribution of electric power,Forestry,655.2187308
Distribution of electric power,Aquaculture,12182.39344
Distribution of electric power,Extractive fishing,59.28061118
Distribution of electric power,Coal mining,497.0991407
Distribution of electric power,Crude oil and gas mining,71.58705547
Distribution of electric power,Copper mining,50937.74365
Distribution of electric power,Iron mining,853.7927901
Distribution of electric power,Mining of other metals,954.907714
Distribution of electric power,Other mining activities and mining services,578.6741561
Distribution of electric power,Processing and preserving of meat,7408.864862
Distribution of electric power,Processing and preserving of fishmeal and fish oil,1192.171449
Distribution of electric power,Processing and preserving of fish,4423.293628
Distribution of electric power,Processing and preserving of fruits and vegetables,4506.917505
Distribution of electric power,Manufacture of vegetable and animal oils,4349.285008
Distribution of electric power,Manufacture of dairy products,5512.906693
Distribution of electric power,Manufacture of grain mill products,2627.914477
Distribution of electric power,Manufacture of prepared animal feeds,13500.68559
Distribution of electric power,Manufacture of bakery products,15074.036
Distribution of electric power,Manufacture of macaroni noodles and other related products,537.3226507
Distribution of electric power,Manufacture of other food products,5835.484036
Distribution of electric power,Distilling rectifying and blending of spirits,1578.043769
Distribution of electric power,Manufacture of wines,14228.12383
Distribution of electric power,Manufacture of beers,4153.273747
Distribution of electric power,Manufacture of soft drinks,10149.34912
Distribution of electric power,Manufacture of tobacco products,1637.307957
Distribution of electric power,Manufacture of textile products,10255.61999
Distribution of electric power,Manufacture of wearing apparel,4319.729055
Distribution of electric power,Manufacture of leather products,750.0478912
Distribution of electric power,Manufacture of footwear,1326.726334
Distribution of electric power,Sawmilling and planing of wood,7770.100042
Distribution of electric power,Manufacture of wood products,11245.42853
Distribution of electric power,Manufacture of pulp and paper,21794.95112
Distribution of electric power,Manufacture of containers of paper,1487.653356
Distribution of electric power,Manufacture of other paper products,4582.051868
Distribution of electric power,Printing,11899.80447
Distribution of electric power,Manufacture of refined petroleum products,8386.695753
Distribution of electric power,Manufacture of basic chemicals,28172.98561
Distribution of electric power,Manufacture of paints,1189.799082
Distribution of electric power,Manufacture of pharmaceutical products,5914.559962
Distribution of electric power,Manufacture of soap detergents and toilet products,2035.291467
Distribution of electric power,Manufacture of other chemical products,2615.812333
Distribution of electric power,Manufacture of rubber products,1093.813124
Distribution of electric power,Manufacture of plastic products,16036.72232
Distribution of electric power,Manufacture of glass and glass products,4759.334039
Distribution of electric power,Manufacture of cement,1791.486981
Distribution of electric power,Manufacture of concrete and concrete products,13821.09243
Distribution of electric power,Manufacture of basic iron and steel,6239.955892
Distribution of electric power,Manufacture of other basic metals,1131.995516
Distribution of electric power,Manufacture of fabricated metal products,14079.37713
Distribution of electric power,Manufacture of industrial and domestic machinery and equipment,5045.364774
Distribution of electric power,Manufacture of electric and electronic machinery and equipment,6438.567368
Distribution of electric power,Manufacture of transport equipment,108.4273515
Distribution of electric power,Manufacture of furniture,2121.056655
Distribution of electric power,Repair and installation of machinery and other manufacturing,10252.53136
Distribution of electric power,Electric power generation,52448.60173
Distribution of electric power,Transmission of electric power,7537.078185
Distribution of electric power,Distribution of electric power,57184.2594
Distribution of electric power,Gas and steam manufacture and supply,2526.326664
Distribution of electric power,Water collection treatment and supply,67849.55064
Distribution of electric power,Waste collection and recycling activities,5752.182239
Distribution of electric power,Construction of residential buildings,22182.72804
Distribution of electric power,Construction of non-residential buildings,10261.03751
Distribution of electric power,Civil engineering,9666.419164
Distribution of electric power,Specialized construction activities,29964.96753
Distribution of electric power,Wholesale and retail trade of motor vehicles,17667.56608
Distribution of electric power,Wholesale trade,75667.6866
Distribution of electric power,Retail trade,230242.6574
Distribution of electric power,Accommodation,22976.44093
Distribution of electric power,Food and beverage service activities,48327.66305
Distribution of electric power,Transport via railways,5614.628138
Distribution of electric power,Other passenger land transport,41827.59458
Distribution of electric power,Freight transport by road,12378.02822
Distribution of electric power,Transport via pipeline,2433.290533
Distribution of electric power,Water transport,370.9733106
Distribution of electric power,Air transport,3582.241124
Distribution of electric power,Warehousing,16448.34291
Distribution of electric power,Support activities for land transport,18504.58523
Distribution of electric power,Other support activities for transport,25972.45851
Distribution of electric power,Postal and courier activities,1098.34029
Distribution of electric power,Wireless telecommunications,27485.79062
Distribution of electric power,Wired telecommunications,33730.10547
Distribution of electric power,Other telecommunications activities,11413.80326
Distribution of electric power,Information services activities,10045.25519
Distribution of electric power,Publishing activities,9116.035049
Distribution of electric power,Banking monetary intermediation,34211.05168
Distribution of electric power,Insurance activities,3244.821027
Distribution of electric power,Auxiliary financial activities,7183.276163
Distribution of electric power,Real estate activities,74963.33759
Distribution of electric power,Housing services,6168.993225
Distribution of electric power,Legal and accounting services,8044.429735
Distribution of electric power,Architectural and engineering services,25758.88504
Distribution of electric power,Other professional activities,23629.79921
Distribution of electric power,Rental and leasing activities,6345.310175
Distribution of electric power,Business support activities,35488.91923
Distribution of electric power,Public administration,183288.2662
Distribution of electric power,Public education,75699.09809
Distribution of electric power,Private education,41657.59824
Distribution of electric power,Public health activities,43471.16018
Distribution of electric power,Private health activities,87180.86313
Distribution of electric power,Activities of organizations,7707.91468
Distribution of electric power,Artistic and entertainment activities,16598.36736
Distribution of electric power,Other personal services,1266.001042
Gas and steam manufacture and supply,Cultivation of annual crops,258.7300843
Gas and steam manufacture and supply,Cultivation of vegetables,80.93243631
Gas and steam manufacture and supply,Cultivation of grapes,139.3894921
Gas and steam manufacture and supply,Cultivation of other fruits,356.2669676
Gas and steam manufacture and supply,Cattle bredding,247.9993529
Gas and steam manufacture and supply,Pigs breeding,83.67473133
Gas and steam manufacture and supply,Chicken breeding,147.37593
Gas and steam manufacture and supply,Breeding of other animals,24.36141559
Gas and steam manufacture and supply,Farming services,48.08833461
Gas and steam manufacture and supply,Forestry,255.1486138
Gas and steam manufacture and supply,Aquaculture,223.7477929
Gas and steam manufacture and supply,Extractive fishing,21.68162184
Gas and steam manufacture and supply,Coal mining,6.616862706
Gas and steam manufacture and supply,Crude oil and gas mining,50.74419854
Gas and steam manufacture and supply,Copper mining,43748.32588
Gas and steam manufacture and supply,Iron mining,96.246874
Gas and steam manufacture and supply,Mining of other metals,313.2458433
Gas and steam manufacture and supply,Other mining activities and mining services,109.9893492
Gas and steam manufacture and supply,Processing and preserving of meat,7707.014517
Gas and steam manufacture and supply,Processing and preserving of fishmeal and fish oil,145.4080542
Gas and steam manufacture and supply,Processing and preserving of fish,2493.194165
Gas and steam manufacture and supply,Processing and preserving of fruits and vegetables,4535.784872
Gas and steam manufacture and supply,Manufacture of vegetable and animal oils,727.7470364
Gas and steam manufacture and supply,Manufacture of dairy products,5161.190086
Gas and steam manufacture and supply,Manufacture of grain mill products,942.6832525
Gas and steam manufacture and supply,Manufacture of prepared animal feeds,2671.494216
Gas and steam manufacture and supply,Manufacture of bakery products,9846.787957
Gas and steam manufacture and supply,Manufacture of macaroni noodles and other related products,557.0184921
Gas and steam manufacture and supply,Manufacture of other food products,3355.173529
Gas and steam manufacture and supply,Distilling rectifying and blending of spirits,532.4253564
Gas and steam manufacture and supply,Manufacture of wines,3902.085777
Gas and steam manufacture and supply,Manufacture of beers,6425.002461
Gas and steam manufacture and supply,Manufacture of soft drinks,4050.094271
Gas and steam manufacture and supply,Manufacture of tobacco products,999.8887804
Gas and steam manufacture and supply,Manufacture of textile products,1893.948833
Gas and steam manufacture and supply,Manufacture of wearing apparel,2510.395978
Gas and steam manufacture and supply,Manufacture of leather products,195.2741911
Gas and steam manufacture and supply,Manufacture of footwear,125.1825385
Gas and steam manufacture and supply,Sawmilling and planing of wood,3165.650132
Gas and steam manufacture and supply,Manufacture of wood products,2112.404104
Gas and steam manufacture and supply,Manufacture of pulp and paper,11239.20744
Gas and steam manufacture and supply,Manufacture of containers of paper,2638.748109
Gas and steam manufacture and supply,Manufacture of other paper products,4305.281766
Gas and steam manufacture and supply,Printing,2552.594893
Gas and steam manufacture and supply,Manufacture of refined petroleum products,85862.76486
Gas and steam manufacture and supply,Manufacture of basic chemicals,48619.90001
Gas and steam manufacture and supply,Manufacture of paints,607.5777505
Gas and steam manufacture and supply,Manufacture of pharmaceutical products,4413.157304
Gas and steam manufacture and supply,Manufacture of soap detergents and toilet products,2839.165082
Gas and steam manufacture and supply,Manufacture of other chemical products,1720.6859
Gas and steam manufacture and supply,Manufacture of rubber products,399.5080437
Gas and steam manufacture and supply,Manufacture of plastic products,3373.300996
Gas and steam manufacture and supply,Manufacture of glass and glass products,9038.750562
Gas and steam manufacture and supply,Manufacture of cement,5964.128635
Gas and steam manufacture and supply,Manufacture of concrete and concrete products,9834.081434
Gas and steam manufacture and supply,Manufacture of basic iron and steel,7406.884802
Gas and steam manufacture and supply,Manufacture of other basic metals,1763.158685
Gas and steam manufacture and supply,Manufacture of fabricated metal products,7863.699709
Gas and steam manufacture and supply,Manufacture of industrial and domestic machinery and equipment,4401.175162
Gas and steam manufacture and supply,Manufacture of electric and electronic machinery and equipment,1821.680018
Gas and steam manufacture and supply,Manufacture of transport equipment,992.1216062
Gas and steam manufacture and supply,Manufacture of furniture,1255.895591
Gas and steam manufacture and supply,Repair and installation of machinery and other manufacturing,4650.955729
Gas and steam manufacture and supply,Electric power generation,407873.3217
Gas and steam manufacture and supply,Transmission of electric power,21.41565597
Gas and steam manufacture and supply,Distribution of electric power,2141.658461
Gas and steam manufacture and supply,Gas and steam manufacture and supply,201517.828
Gas and steam manufacture and supply,Water collection treatment and supply,186.2135405
Gas and steam manufacture and supply,Waste collection and recycling activities,428.6952644
Gas and steam manufacture and supply,Construction of residential buildings,3913.262753
Gas and steam manufacture and supply,Construction of non-residential buildings,1465.130372
Gas and steam manufacture and supply,Civil engineering,1141.808152
Gas and steam manufacture and supply,Specialized construction activities,461.7361896
Gas and steam manufacture and supply,Wholesale and retail trade of motor vehicles,3524.126167
Gas and steam manufacture and supply,Wholesale trade,21393.15223
Gas and steam manufacture and supply,Retail trade,17494.39901
Gas and steam manufacture and supply,Accommodation,4312.848913
Gas and steam manufacture and supply,Food and beverage service activities,16893.23455
Gas and steam manufacture and supply,Transport via railways,190.0085985
Gas and steam manufacture and supply,Other passenger land transport,7394.302257
Gas and steam manufacture and supply,Freight transport by road,3661.176309
Gas and steam manufacture and supply,Transport via pipeline,5.188636549
Gas and steam manufacture and supply,Water transport,211.473204
Gas and steam manufacture and supply,Air transport,1168.34229
Gas and steam manufacture and supply,Warehousing,870.3492713
Gas and steam manufacture and supply,Support activities for land transport,758.7066779
Gas and steam manufacture and supply,Other support activities for transport,1183.523705
Gas and steam manufacture and supply,Postal and courier activities,62.23958583
Gas and steam manufacture and supply,Wireless telecommunications,2268.177126
Gas and steam manufacture and supply,Wired telecommunications,631.0145075
Gas and steam manufacture and supply,Other telecommunications activities,2734.768983
Gas and steam manufacture and supply,Information services activities,3559.859349
Gas and steam manufacture and supply,Publishing activities,2013.573587
Gas and steam manufacture and supply,Banking monetary intermediation,6823.59399
Gas and steam manufacture and supply,Insurance activities,4415.863837
Gas and steam manufacture and supply,Auxiliary financial activities,1798.097936
Gas and steam manufacture and supply,Real estate activities,6279.92547
Gas and steam manufacture and supply,Housing services,0
Gas and steam manufacture and supply,Legal and accounting services,2817.588728
Gas and steam manufacture and supply,Architectural and engineering services,6372.640843
Gas and steam manufacture and supply,Other professional activities,11780.55755
Gas and steam manufacture and supply,Rental and leasing activities,2540.933847
Gas and steam manufacture and supply,Business support activities,9811.415261
Gas and steam manufacture and supply,Public administration,2992.926089
Gas and steam manufacture and supply,Public education,7107.385059
Gas and steam manufacture and supply,Private education,6102.451926
Gas and steam manufacture and supply,Public health activities,3191.370854
Gas and steam manufacture and supply,Private health activities,3957.979939
Gas and steam manufacture and supply,Activities of organizations,406.6461791
Gas and steam manufacture and supply,Artistic and entertainment activities,2186.034189
Gas and steam manufacture and supply,Other personal services,869.4439374
Water collection treatment and supply,Cultivation of annual crops,359.5297175
Water collection treatment and supply,Cultivation of vegetables,203.2042529
Water collection treatment and supply,Cultivation of grapes,457.8165441
Water collection treatment and supply,Cultivation of other fruits,877.267718
Water collection treatment and supply,Cattle bredding,354.0908968
Water collection treatment and supply,Pigs breeding,62.36663458
Water collection treatment and supply,Chicken breeding,92.23362973
Water collection treatment and supply,Breeding of other animals,17.36482311
Water collection treatment and supply,Farming services,174.4516377
Water collection treatment and supply,Forestry,0.688457304
Water collection treatment and supply,Aquaculture,381.6503598
Water collection treatment and supply,Extractive fishing,99.66071017
Water collection treatment and supply,Coal mining,1.236984984
Water collection treatment and supply,Crude oil and gas mining,1.956564088
Water collection treatment and supply,Copper mining,14798.73987
Water collection treatment and supply,Iron mining,1843.91552
Water collection treatment and supply,Mining of other metals,525.3839628
Water collection treatment and supply,Other mining activities and mining services,3139.094848
Water collection treatment and supply,Processing and preserving of meat,1329.741332
Water collection treatment and supply,Processing and preserving of fishmeal and fish oil,847.22513
Water collection treatment and supply,Processing and preserving of fish,3814.927081
Water collection treatment and supply,Processing and preserving of fruits and vegetables,1036.802461
Water collection treatment and supply,Manufacture of vegetable and animal oils,485.0833368
Water collection treatment and supply,Manufacture of dairy products,2446.489489
Water collection treatment and supply,Manufacture of grain mill products,286.2507457
Water collection treatment and supply,Manufacture of prepared animal feeds,229.6402819
Water collection treatment and supply,Manufacture of bakery products,3437.126104
Water collection treatment and supply,Manufacture of macaroni noodles and other related products,89.42118275
Water collection treatment and supply,Manufacture of other food products,1476.815906
Water collection treatment and supply,Distilling rectifying and blending of spirits,81.87236434
Water collection treatment and supply,Manufacture of wines,739.8374069
Water collection treatment and supply,Manufacture of beers,4319.443918
Water collection treatment and supply,Manufacture of soft drinks,4088.551514
Water collection treatment and supply,Manufacture of tobacco products,3.153512578
Water collection treatment and supply,Manufacture of textile products,1645.613953
Water collection treatment and supply,Manufacture of wearing apparel,1034.530399
Water collection treatment and supply,Manufacture of leather products,230.9785082
Water collection treatment and supply,Manufacture of footwear,298.3167084
Water collection treatment and supply,Sawmilling and planing of wood,272.6310258
Water collection treatment and supply,Manufacture of wood products,363.371782
Water collection treatment and supply,Manufacture of pulp and paper,1529.921951
Water collection treatment and supply,Manufacture of containers of paper,414.9630198
Water collection treatment and supply,Manufacture of other paper products,291.2105561
Water collection treatment and supply,Printing,605.6350917
Water collection treatment and supply,Manufacture of refined petroleum products,410.4292464
Water collection treatment and supply,Manufacture of basic chemicals,4825.942821
Water collection treatment and supply,Manufacture of paints,230.4058086
Water collection treatment and supply,Manufacture of pharmaceutical products,895.0798311
Water collection treatment and supply,Manufacture of soap detergents and toilet products,793.0580575
Water collection treatment and supply,Manufacture of other chemical products,1343.324936
Water collection treatment and supply,Manufacture of rubber products,387.6710837
Water collection treatment and supply,Manufacture of plastic products,1771.024315
Water collection treatment and supply,Manufacture of glass and glass products,355.1651515
Water collection treatment and supply,Manufacture of cement,571.1802321
Water collection treatment and supply,Manufacture of concrete and concrete products,2625.362308
Water collection treatment and supply,Manufacture of basic iron and steel,727.6582318
Water collection treatment and supply,Manufacture of other basic metals,263.9026552
Water collection treatment and supply,Manufacture of fabricated metal products,2395.707595
Water collection treatment and supply,Manufacture of industrial and domestic machinery and equipment,2243.402865
Water collection treatment and supply,Manufacture of electric and electronic machinery and equipment,911.8264955
Water collection treatment and supply,Manufacture of transport equipment,474.5410929
Water collection treatment and supply,Manufacture of furniture,835.9639938
Water collection treatment and supply,Repair and installation of machinery and other manufacturing,4071.2928
Water collection treatment and supply,Electric power generation,9.36445842
Water collection treatment and supply,Transmission of electric power,104.9036883
Water collection treatment and supply,Distribution of electric power,227.291646
Water collection treatment and supply,Gas and steam manufacture and supply,222.1085783
Water collection treatment and supply,Water collection treatment and supply,45841.295
Water collection treatment and supply,Waste collection and recycling activities,974.0342983
Water collection treatment and supply,Construction of residential buildings,2078.66153
Water collection treatment and supply,Construction of non-residential buildings,941.4035077
Water collection treatment and supply,Civil engineering,6400.636712
Water collection treatment and supply,Specialized construction activities,9996.031114
Water collection treatment and supply,Wholesale and retail trade of motor vehicles,3506.962176
Water collection treatment and supply,Wholesale trade,11028.3955
Water collection treatment and supply,Retail trade,16356.31913
Water collection treatment and supply,Accommodation,5178.617594
Water collection treatment and supply,Food and beverage service activities,14540.52448
Water collection treatment and supply,Transport via railways,536.7413497
Water collection treatment and supply,Other passenger land transport,2527.233404
Water collection treatment and supply,Freight transport by road,3620.859547
Water collection treatment and supply,Transport via pipeline,32.21718807
Water collection treatment and supply,Water transport,102.019491
Water collection treatment and supply,Air transport,213.8520826
Water collection treatment and supply,Warehousing,704.1536761
Water collection treatment and supply,Support activities for land transport,907.2407034
Water collection treatment and supply,Other support activities for transport,3642.935614
Water collection treatment and supply,Postal and courier activities,2.552423217
Water collection treatment and supply,Wireless telecommunications,288.8022705
Water collection treatment and supply,Wired telecommunications,379.8050594
Water collection treatment and supply,Other telecommunications activities,18.69742855
Water collection treatment and supply,Information services activities,1657.892064
Water collection treatment and supply,Publishing activities,893.3759054
Water collection treatment and supply,Banking monetary intermediation,2085.98039
Water collection treatment and supply,Insurance activities,1065.931011
Water collection treatment and supply,Auxiliary financial activities,828.7850343
Water collection treatment and supply,Real estate activities,6206.030971
Water collection treatment and supply,Housing services,0
Water collection treatment and supply,Legal and accounting services,1514.045942
Water collection treatment and supply,Architectural and engineering services,5340.190751
Water collection treatment and supply,Other professional activities,5165.10565
Water collection treatment and supply,Rental and leasing activities,1878.015081
Water collection treatment and supply,Business support activities,1389.698188
Water collection treatment and supply,Public administration,38033.79944
Water collection treatment and supply,Public education,27381.54745
Water collection treatment and supply,Private education,14984.4659
Water collection treatment and supply,Public health activities,14324.88085
Water collection treatment and supply,Private health activities,11041.0386
Water collection treatment and supply,Activities of organizations,4103.845578
Water collection treatment and supply,Artistic and entertainment activities,3833.238165
Water collection treatment and supply,Other personal services,8487.201348
Waste collection and recycling activities,Cultivation of annual crops,88.0769264
Waste collection and recycling activities,Cultivation of vegetables,48.13758986
Waste collection and recycling activities,Cultivation of grapes,51.79991336
Waste collection and recycling activities,Cultivation of other fruits,174.7978536
Waste collection and recycling activities,Cattle bredding,155.9137863
Waste collection and recycling activities,Pigs breeding,69.55180301
Waste collection and recycling activities,Chicken breeding,141.8911238
Waste collection and recycling activities,Breeding of other animals,15.68004269
Waste collection and recycling activities,Farming services,1.918670815
Waste collection and recycling activities,Forestry,2.317220899
Waste collection and recycling activities,Aquaculture,644.0128851
Waste collection and recycling activities,Extractive fishing,0.73661997
Waste collection and recycling activities,Coal mining,175.9914464
Waste collection and recycling activities,Crude oil and gas mining,43.38180441
Waste collection and recycling activities,Copper mining,30102.06941
Waste collection and recycling activities,Iron mining,187.6567938
Waste collection and recycling activities,Mining of other metals,1435.767993
Waste collection and recycling activities,Other mining activities and mining services,68.1804596
Waste collection and recycling activities,Processing and preserving of meat,3597.985161
Waste collection and recycling activities,Processing and preserving of fishmeal and fish oil,270.8410502
Waste collection and recycling activities,Processing and preserving of fish,3845.958307
Waste collection and recycling activities,Processing and preserving of fruits and vegetables,235.9522095
Waste collection and recycling activities,Manufacture of vegetable and animal oils,475.7364503
Waste collection and recycling activities,Manufacture of dairy products,3274.559193
Waste collection and recycling activities,Manufacture of grain mill products,620.8256152
Waste collection and recycling activities,Manufacture of prepared animal feeds,3332.330764
Waste collection and recycling activities,Manufacture of bakery products,94.46976366
Waste collection and recycling activities,Manufacture of macaroni noodles and other related products,92.94388053
Waste collection and recycling activities,Manufacture of other food products,589.1178206
Waste collection and recycling activities,Distilling rectifying and blending of spirits,255.3703881
Waste collection and recycling activities,Manufacture of wines,268.957066
Waste collection and recycling activities,Manufacture of beers,805.7020647
Waste collection and recycling activities,Manufacture of soft drinks,613.4102202
Waste collection and recycling activities,Manufacture of tobacco products,204.6694435
Waste collection and recycling activities,Manufacture of textile products,76.19956446
Waste collection and recycling activities,Manufacture of wearing apparel,85.37797984
Waste collection and recycling activities,Manufacture of leather products,194.0200887
Waste collection and recycling activities,Manufacture of footwear,536.9455818
Waste collection and recycling activities,Sawmilling and planing of wood,628.167193
Waste collection and recycling activities,Manufacture of wood products,720.7846048
Waste collection and recycling activities,Manufacture of pulp and paper,41428.33715
Waste collection and recycling activities,Manufacture of containers of paper,823.1449412
Waste collection and recycling activities,Manufacture of other paper products,58956.29242
Waste collection and recycling activities,Printing,202.1563137
Waste collection and recycling activities,Manufacture of refined petroleum products,1207.275812
Waste collection and recycling activities,Manufacture of basic chemicals,5949.825422
Waste collection and recycling activities,Manufacture of paints,202.3457011
Waste collection and recycling activities,Manufacture of pharmaceutical products,860.1079147
Waste collection and recycling activities,Manufacture of soap detergents and toilet products,495.9088275
Waste collection and recycling activities,Manufacture of other chemical products,4.702818842
Waste collection and recycling activities,Manufacture of rubber products,575.6470121
Waste collection and recycling activities,Manufacture of plastic products,27810.45271
Waste collection and recycling activities,Manufacture of glass and glass products,604.3307306
Waste collection and recycling activities,Manufacture of cement,403.0178251
Waste collection and recycling activities,Manufacture of concrete and concrete products,796.5840635
Waste collection and recycling activities,Manufacture of basic iron and steel,1277.046704
Waste collection and recycling activities,Manufacture of other basic metals,589.4254245
Waste collection and recycling activities,Manufacture of fabricated metal products,700.1693151
Waste collection and recycling activities,Manufacture of industrial and domestic machinery and equipment,35.23990166
Waste collection and recycling activities,Manufacture of electric and electronic machinery and equipment,59.82667572
Waste collection and recycling activities,Manufacture of transport equipment,309.6498212
Waste collection and recycling activities,Manufacture of furniture,269.5524881
Waste collection and recycling activities,Repair and installation of machinery and other manufacturing,291.7210399
Waste collection and recycling activities,Electric power generation,807.7742415
Waste collection and recycling activities,Transmission of electric power,39.58649544
Waste collection and recycling activities,Distribution of electric power,5584.256247
Waste collection and recycling activities,Gas and steam manufacture and supply,40.30675523
Waste collection and recycling activities,Water collection treatment and supply,59625.92189
Waste collection and recycling activities,Waste collection and recycling activities,16979.1166
Waste collection and recycling activities,Construction of residential buildings,443.2214187
Waste collection and recycling activities,Construction of non-residential buildings,293.2030184
Waste collection and recycling activities,Civil engineering,448.2741292
Waste collection and recycling activities,Specialized construction activities,284.7014012
Waste collection and recycling activities,Wholesale and retail trade of motor vehicles,597.4855167
Waste collection and recycling activities,Wholesale trade,3327.151282
Waste collection and recycling activities,Retail trade,1668.770989
Waste collection and recycling activities,Accommodation,33.57252982
Waste collection and recycling activities,Food and beverage service activities,384.7926737
Waste collection and recycling activities,Transport via railways,1.411071051
Waste collection and recycling activities,Other passenger land transport,52.56850081
Waste collection and recycling activities,Freight transport by road,441.3300663
Waste collection and recycling activities,Transport via pipeline,12.94888929
Waste collection and recycling activities,Water transport,1.801276564
Waste collection and recycling activities,Air transport,85.32220904
Waste collection and recycling activities,Warehousing,157.2954892
Waste collection and recycling activities,Support activities for land transport,192.1736138
Waste collection and recycling activities,Other support activities for transport,273.8917899
Waste collection and recycling activities,Postal and courier activities,0.34282687
Waste collection and recycling activities,Wireless telecommunications,144.7438203
Waste collection and recycling activities,Wired telecommunications,79.98232298
Waste collection and recycling activities,Other telecommunications activities,16.6524253
Waste collection and recycling activities,Information services activities,423.1708171
Waste collection and recycling activities,Publishing activities,273.7614126
Waste collection and recycling activities,Banking monetary intermediation,149.6273545
Waste collection and recycling activities,Insurance activities,44.07097807
Waste collection and recycling activities,Auxiliary financial activities,34.47887348
Waste collection and recycling activities,Real estate activities,21.88882994
Waste collection and recycling activities,Housing services,0
Waste collection and recycling activities,Legal and accounting services,1719.147098
Waste collection and recycling activities,Architectural and engineering services,2674.303269
Waste collection and recycling activities,Other professional activities,2747.36818
Waste collection and recycling activities,Rental and leasing activities,17.95783351
Waste collection and recycling activities,Business support activities,1338.948415
Waste collection and recycling activities,Public administration,172987.7448
Waste collection and recycling activities,Public education,188.8960279
Waste collection and recycling activities,Private education,42.99009836
Waste collection and recycling activities,Public health activities,102.1527778
Waste collection and recycling activities,Private health activities,3985.351535
Waste collection and recycling activities,Activities of organizations,24.43204919
Waste collection and recycling activities,Artistic and entertainment activities,313.4846799
Waste collection and recycling activities,Other personal services,255.9169837
Construction of residential buildings,Cultivation of annual crops,146.1235638
Construction of residential buildings,Cultivation of vegetables,81.16953384
Construction of residential buildings,Cultivation of grapes,142.0219438
Construction of residential buildings,Cultivation of other fruits,457.7147329
Construction of residential buildings,Cattle bredding,219.301528
Construction of residential buildings,Pigs breeding,112.9092446
Construction of residential buildings,Chicken breeding,219.0844891
Construction of residential buildings,Breeding of other animals,24.18758355
Construction of residential buildings,Farming services,5.881502469
Construction of residential buildings,Forestry,199.8526705
Construction of residential buildings,Aquaculture,248.5463451
Construction of residential buildings,Extractive fishing,168.3816905
Construction of residential buildings,Coal mining,0.405361047
Construction of residential buildings,Crude oil and gas mining,54.00248347
Construction of residential buildings,Copper mining,1694.278881
Construction of residential buildings,Iron mining,350.6646838
Construction of residential buildings,Mining of other metals,123.210774
Construction of residential buildings,Other mining activities and mining services,166.297902
Construction of residential buildings,Processing and preserving of meat,183.7408578
Construction of residential buildings,Processing and preserving of fishmeal and fish oil,1.173937705
Construction of residential buildings,Processing and preserving of fish,71.6033225
Construction of residential buildings,Processing and preserving of fruits and vegetables,136.2735253
Construction of residential buildings,Manufacture of vegetable and animal oils,27.48679589
Construction of residential buildings,Manufacture of dairy products,306.718357
Construction of residential buildings,Manufacture of grain mill products,104.707822
Construction of residential buildings,Manufacture of prepared animal feeds,133.2006107
Construction of residential buildings,Manufacture of bakery products,496.5666344
Construction of residential buildings,Manufacture of macaroni noodles and other related products,64.61723482
Construction of residential buildings,Manufacture of other food products,241.8831413
Construction of residential buildings,Distilling rectifying and blending of spirits,30.58648759
Construction of residential buildings,Manufacture of wines,352.2842051
Construction of residential buildings,Manufacture of beers,42.50509451
Construction of residential buildings,Manufacture of soft drinks,250.9525199
Construction of residential buildings,Manufacture of tobacco products,2.342785213
Construction of residential buildings,Manufacture of textile products,204.6247241
Construction of residential buildings,Manufacture of wearing apparel,489.2327402
Construction of residential buildings,Manufacture of leather products,21.03478561
Construction of residential buildings,Manufacture of footwear,110.7705279
Construction of residential buildings,Sawmilling and planing of wood,211.6635931
Construction of residential buildings,Manufacture of wood products,133.654491
Construction of residential buildings,Manufacture of pulp and paper,132.5625723
Construction of residential buildings,Manufacture of containers of paper,86.13528281
Construction of residential buildings,Manufacture of other paper products,288.9255127
Construction of residential buildings,Printing,241.0481242
Construction of residential buildings,Manufacture of refined petroleum products,165.7404171
Construction of residential buildings,Manufacture of basic chemicals,146.3265428
Construction of residential buildings,Manufacture of paints,86.91688574
Construction of residential buildings,Manufacture of pharmaceutical products,348.9467195
Construction of residential buildings,Manufacture of soap detergents and toilet products,181.9302403
Construction of residential buildings,Manufacture of other chemical products,140.1277902
Construction of residential buildings,Manufacture of rubber products,34.5188145
Construction of residential buildings,Manufacture of plastic products,346.2219445
Construction of residential buildings,Manufacture of glass and glass products,87.61007809
Construction of residential buildings,Manufacture of cement,187.4304713
Construction of residential buildings,Manufacture of concrete and concrete products,373.4508223
Construction of residential buildings,Manufacture of basic iron and steel,40.65757768
Construction of residential buildings,Manufacture of other basic metals,28.8594285
Construction of residential buildings,Manufacture of fabricated metal products,567.5397261
Construction of residential buildings,Manufacture of industrial and domestic machinery and equipment,206.4489312
Construction of residential buildings,Manufacture of electric and electronic machinery and equipment,239.5907162
Construction of residential buildings,Manufacture of transport equipment,136.9306217
Construction of residential buildings,Manufacture of furniture,409.2153689
Construction of residential buildings,Repair and installation of machinery and other manufacturing,568.5289952
Construction of residential buildings,Electric power generation,668.1663814
Construction of residential buildings,Transmission of electric power,49.27689239
Construction of residential buildings,Distribution of electric power,884.7304697
Construction of residential buildings,Gas and steam manufacture and supply,185.6350376
Construction of residential buildings,Water collection treatment and supply,2753.651993
Construction of residential buildings,Waste collection and recycling activities,228.2109195
Construction of residential buildings,Construction of residential buildings,59412.85179
Construction of residential buildings,Construction of non-residential buildings,46773.99429
Construction of residential buildings,Civil engineering,13597.82624
Construction of residential buildings,Specialized construction activities,99.50223244
Construction of residential buildings,Wholesale and retail trade of motor vehicles,2707.564523
Construction of residential buildings,Wholesale trade,10987.46602
Construction of residential buildings,Retail trade,25128.81693
Construction of residential buildings,Accommodation,2821.534755
Construction of residential buildings,Food and beverage service activities,4256.113825
Construction of residential buildings,Transport via railways,415.7199616
Construction of residential buildings,Other passenger land transport,829.1633302
Construction of residential buildings,Freight transport by road,1719.546324
Construction of residential buildings,Transport via pipeline,19.68136647
Construction of residential buildings,Water transport,21.58595762
Construction of residential buildings,Air transport,1849.971953
Construction of residential buildings,Warehousing,1181.279573
Construction of residential buildings,Support activities for land transport,3024.981443
Construction of residential buildings,Other support activities for transport,1207.032827
Construction of residential buildings,Postal and courier activities,80.22757522
Construction of residential buildings,Wireless telecommunications,3054.211654
Construction of residential buildings,Wired telecommunications,1373.160939
Construction of residential buildings,Other telecommunications activities,196.2312783
Construction of residential buildings,Information services activities,2642.738625
Construction of residential buildings,Publishing activities,1212.350774
Construction of residential buildings,Banking monetary intermediation,3041.870374
Construction of residential buildings,Insurance activities,792.1929993
Construction of residential buildings,Auxiliary financial activities,574.6044439
Construction of residential buildings,Real estate activities,8226.516976
Construction of residential buildings,Housing services,79049.0143
Construction of residential buildings,Legal and accounting services,1581.013917
Construction of residential buildings,Architectural and engineering services,3990.008842
Construction of residential buildings,Other professional activities,4044.30828
Construction of residential buildings,Rental and leasing activities,1423.033173
Construction of residential buildings,Business support activities,4339.096978
Construction of residential buildings,Public administration,13712.19874
Construction of residential buildings,Public education,8793.791498
Construction of residential buildings,Private education,4385.225498
Construction of residential buildings,Public health activities,1871.143836
Construction of residential buildings,Private health activities,3563.945674
Construction of residential buildings,Activities of organizations,1970.147046
Construction of residential buildings,Artistic and entertainment activities,2156.503922
Construction of residential buildings,Other personal services,1444.67431
Construction of non-residential buildings,Cultivation of annual crops,75.77441819
Construction of non-residential buildings,Cultivation of vegetables,45.50610279
Construction of non-residential buildings,Cultivation of grapes,97.94609838
Construction of non-residential buildings,Cultivation of other fruits,331.1074433
Construction of non-residential buildings,Cattle bredding,177.8903303
Construction of non-residential buildings,Pigs breeding,97.09982367
Construction of non-residential buildings,Chicken breeding,195.6820016
Construction of non-residential buildings,Breeding of other animals,21.96175954
Construction of non-residential buildings,Farming services,2.051728309
Construction of non-residential buildings,Forestry,227.2967404
Construction of non-residential buildings,Aquaculture,71.35461764
Construction of non-residential buildings,Extractive fishing,195.921637
Construction of non-residential buildings,Coal mining,0.304244782
Construction of non-residential buildings,Crude oil and gas mining,57.93872787
Construction of non-residential buildings,Copper mining,835.0935257
Construction of non-residential buildings,Iron mining,135.1092805
Construction of non-residential buildings,Mining of other metals,128.0825277
Construction of non-residential buildings,Other mining activities and mining services,190.2282241
Construction of non-residential buildings,Processing and preserving of meat,106.9423335
Construction of non-residential buildings,Processing and preserving of fishmeal and fish oil,1.099308255
Construction of non-residential buildings,Processing and preserving of fish,62.47226155
Construction of non-residential buildings,Processing and preserving of fruits and vegetables,88.91479884
Construction of non-residential buildings,Manufacture of vegetable and animal oils,22.03555801
Construction of non-residential buildings,Manufacture of dairy products,200.8967882
Construction of non-residential buildings,Manufacture of grain mill products,73.32025252
Construction of non-residential buildings,Manufacture of prepared animal feeds,93.57800814
Construction of non-residential buildings,Manufacture of bakery products,202.2809361
Construction of non-residential buildings,Manufacture of macaroni noodles and other related products,24.62829699
Construction of non-residential buildings,Manufacture of other food products,149.4038401
Construction of non-residential buildings,Distilling rectifying and blending of spirits,15.60812912
Construction of non-residential buildings,Manufacture of wines,247.9670706
Construction of non-residential buildings,Manufacture of beers,23.14295894
Construction of non-residential buildings,Manufacture of soft drinks,160.0846266
Construction of non-residential buildings,Manufacture of tobacco products,3.147161196
Construction of non-residential buildings,Manufacture of textile products,103.6982647
Construction of non-residential buildings,Manufacture of wearing apparel,190.5843016
Construction of non-residential buildings,Manufacture of leather products,9.629992292
Construction of non-residential buildings,Manufacture of footwear,45.23133518
Construction of non-residential buildings,Sawmilling and planing of wood,122.2613525
Construction of non-residential buildings,Manufacture of wood products,78.693022
Construction of non-residential buildings,Manufacture of pulp and paper,115.2614853
Construction of non-residential buildings,Manufacture of containers of paper,47.53904784
Construction of non-residential buildings,Manufacture of other paper products,170.7817793
Construction of non-residential buildings,Printing,115.4625349
Construction of non-residential buildings,Manufacture of refined petroleum products,136.3543361
Construction of non-residential buildings,Manufacture of basic chemicals,69.85278178
Construction of non-residential buildings,Manufacture of paints,42.3414654
Construction of non-residential buildings,Manufacture of pharmaceutical products,222.2209651
Construction of non-residential buildings,Manufacture of soap detergents and toilet products,128.6039926
Construction of non-residential buildings,Manufacture of other chemical products,44.43052279
Construction of non-residential buildings,Manufacture of rubber products,19.99352692
Construction of non-residential buildings,Manufacture of plastic products,177.6375644
Construction of non-residential buildings,Manufacture of glass and glass products,55.86131753
Construction of non-residential buildings,Manufacture of cement,158.576079
Construction of non-residential buildings,Manufacture of concrete and concrete products,213.6167813
Construction of non-residential buildings,Manufacture of basic iron and steel,13.95942589
Construction of non-residential buildings,Manufacture of other basic metals,22.78700465
Construction of non-residential buildings,Manufacture of fabricated metal products,313.6768671
Construction of non-residential buildings,Manufacture of industrial and domestic machinery and equipment,65.23384338
Construction of non-residential buildings,Manufacture of electric and electronic machinery and equipment,93.98428179
Construction of non-residential buildings,Manufacture of transport equipment,75.09655291
Construction of non-residential buildings,Manufacture of furniture,196.8648038
Construction of non-residential buildings,Repair and installation of machinery and other manufacturing,190.0547796
Construction of non-residential buildings,Electric power generation,598.1715424
Construction of non-residential buildings,Transmission of electric power,17.85055714
Construction of non-residential buildings,Distribution of electric power,867.9132624
Construction of non-residential buildings,Gas and steam manufacture and supply,189.8156256
Construction of non-residential buildings,Water collection treatment and supply,3130.361329
Construction of non-residential buildings,Waste collection and recycling activities,148.7675008
Construction of non-residential buildings,Construction of residential buildings,68906.92261
Construction of non-residential buildings,Construction of non-residential buildings,54440.99733
Construction of non-residential buildings,Civil engineering,15510.78267
Construction of non-residential buildings,Specialized construction activities,32.2573875
Construction of non-residential buildings,Wholesale and retail trade of motor vehicles,1458.39518
Construction of non-residential buildings,Wholesale trade,6611.016469
Construction of non-residential buildings,Retail trade,10813.25438
Construction of non-residential buildings,Accommodation,1449.411768
Construction of non-residential buildings,Food and beverage service activities,1766.311103
Construction of non-residential buildings,Transport via railways,463.1900287
Construction of non-residential buildings,Other passenger land transport,619.1150904
Construction of non-residential buildings,Freight transport by road,1007.190694
Construction of non-residential buildings,Transport via pipeline,6.518315606
Construction of non-residential buildings,Water transport,9.287027924
Construction of non-residential buildings,Air transport,821.3412504
Construction of non-residential buildings,Warehousing,682.0545755
Construction of non-residential buildings,Support activities for land transport,3216.674189
Construction of non-residential buildings,Other support activities for transport,619.8054071
Construction of non-residential buildings,Postal and courier activities,39.29617828
Construction of non-residential buildings,Wireless telecommunications,906.051536
Construction of non-residential buildings,Wired telecommunications,1054.878384
Construction of non-residential buildings,Other telecommunications activities,78.05096628
Construction of non-residential buildings,Information services activities,1357.300397
Construction of non-residential buildings,Publishing activities,482.705042
Construction of non-residential buildings,Banking monetary intermediation,1385.937526
Construction of non-residential buildings,Insurance activities,423.3052174
Construction of non-residential buildings,Auxiliary financial activities,350.5136857
Construction of non-residential buildings,Real estate activities,5355.551546
Construction of non-residential buildings,Housing services,92445.24635
Construction of non-residential buildings,Legal and accounting services,614.8486646
Construction of non-residential buildings,Architectural and engineering services,1824.853867
Construction of non-residential buildings,Other professional activities,1877.615441
Construction of non-residential buildings,Rental and leasing activities,733.8962017
Construction of non-residential buildings,Business support activities,2275.361311
Construction of non-residential buildings,Public administration,14264.20158
Construction of non-residential buildings,Public education,6879.440583
Construction of non-residential buildings,Private education,3015.417197
Construction of non-residential buildings,Public health activities,1815.895192
Construction of non-residential buildings,Private health activities,2183.375021
Construction of non-residential buildings,Activities of organizations,1704.598891
Construction of non-residential buildings,Artistic and entertainment activities,1067.573958
Construction of non-residential buildings,Other personal services,1083.021077
Civil engineering,Cultivation of annual crops,44.55270629
Civil engineering,Cultivation of vegetables,29.12193017
Civil engineering,Cultivation of grapes,73.21789654
Civil engineering,Cultivation of other fruits,249.8866111
Civil engineering,Cattle bredding,142.7873991
Civil engineering,Pigs breeding,79.19418643
Civil engineering,Chicken breeding,162.1633137
Civil engineering,Breeding of other animals,18.47061639
Civil engineering,Farming services,1.744749193
Civil engineering,Forestry,208.9243305
Civil engineering,Aquaculture,0.99779031
Civil engineering,Extractive fishing,179.1944181
Civil engineering,Coal mining,0.24627521
Civil engineering,Crude oil and gas mining,53.81636647
Civil engineering,Copper mining,1076.578901
Civil engineering,Iron mining,42.32302731
Civil engineering,Mining of other metals,113.6524096
Civil engineering,Other mining activities and mining services,173.6569152
Civil engineering,Processing and preserving of meat,113.4096559
Civil engineering,Processing and preserving of fishmeal and fish oil,3.306522322
Civil engineering,Processing and preserving of fish,71.59337876
Civil engineering,Processing and preserving of fruits and vegetables,69.20805414
Civil engineering,Manufacture of vegetable and animal oils,19.77555531
Civil engineering,Manufacture of dairy products,298.2715671
Civil engineering,Manufacture of grain mill products,115.2851198
Civil engineering,Manufacture of prepared animal feeds,107.4029167
Civil engineering,Manufacture of bakery products,180.4173992
Civil engineering,Manufacture of macaroni noodles and other related products,9.05211144
Civil engineering,Manufacture of other food products,232.4373138
Civil engineering,Distilling rectifying and blending of spirits,12.08266691
Civil engineering,Manufacture of wines,331.886024
Civil engineering,Manufacture of beers,49.11931965
Civil engineering,Manufacture of soft drinks,310.7095605
Civil engineering,Manufacture of tobacco products,5.729734621
Civil engineering,Manufacture of textile products,96.52249368
Civil engineering,Manufacture of wearing apparel,185.9999758
Civil engineering,Manufacture of leather products,4.650615214
Civil engineering,Manufacture of footwear,17.87717029
Civil engineering,Sawmilling and planing of wood,182.4066757
Civil engineering,Manufacture of wood products,102.1565297
Civil engineering,Manufacture of pulp and paper,178.2948903
Civil engineering,Manufacture of containers of paper,44.46845714
Civil engineering,Manufacture of other paper products,203.4476132
Civil engineering,Printing,114.5431746
Civil engineering,Manufacture of refined petroleum products,216.9101639
Civil engineering,Manufacture of basic chemicals,130.9638678
Civil engineering,Manufacture of paints,50.96008144
Civil engineering,Manufacture of pharmaceutical products,339.550228
Civil engineering,Manufacture of soap detergents and toilet products,227.7284426
Civil engineering,Manufacture of other chemical products,18.58853161
Civil engineering,Manufacture of rubber products,16.19315354
Civil engineering,Manufacture of plastic products,111.8799757
Civil engineering,Manufacture of glass and glass products,62.64748631
Civil engineering,Manufacture of cement,154.3768509
Civil engineering,Manufacture of concrete and concrete products,215.6213272
Civil engineering,Manufacture of basic iron and steel,9.828977222
Civil engineering,Manufacture of other basic metals,41.14710419
Civil engineering,Manufacture of fabricated metal products,440.2155806
Civil engineering,Manufacture of industrial and domestic machinery and equipment,26.37884211
Civil engineering,Manufacture of electric and electronic machinery and equipment,75.16759799
Civil engineering,Manufacture of transport equipment,77.33966421
Civil engineering,Manufacture of furniture,251.9844139
Civil engineering,Repair and installation of machinery and other manufacturing,51.42076153
Civil engineering,Electric power generation,509.2358871
Civil engineering,Transmission of electric power,8.528678127
Civil engineering,Distribution of electric power,816.9610067
Civil engineering,Gas and steam manufacture and supply,197.0857305
Civil engineering,Water collection treatment and supply,2848.912452
Civil engineering,Waste collection and recycling activities,126.1740215
Civil engineering,Construction of residential buildings,62775.93265
Civil engineering,Construction of non-residential buildings,49638.91533
Civil engineering,Civil engineering,14098.35109
Civil engineering,Specialized construction activities,13.81860013
Civil engineering,Wholesale and retail trade of motor vehicles,924.7624369
Civil engineering,Wholesale trade,4997.341464
Civil engineering,Retail trade,5060.908864
Civil engineering,Accommodation,790.0866514
Civil engineering,Food and beverage service activities,726.3387634
Civil engineering,Transport via railways,425.1984564
Civil engineering,Other passenger land transport,476.5829021
Civil engineering,Freight transport by road,654.3589371
Civil engineering,Transport via pipeline,2.112437989
Civil engineering,Water transport,14.69706646
Civil engineering,Air transport,560.478371
Civil engineering,Warehousing,421.0733637
Civil engineering,Support activities for land transport,2875.998414
Civil engineering,Other support activities for transport,475.8248218
Civil engineering,Postal and courier activities,20.1617347
Civil engineering,Wireless telecommunications,150.0225721
Civil engineering,Wired telecommunications,1115.14906
Civil engineering,Other telecommunications activities,98.39943689
Civil engineering,Information services activities,2319.859733
Civil engineering,Publishing activities,223.8890424
Civil engineering,Banking monetary intermediation,1963.300343
Civil engineering,Insurance activities,545.7462657
Civil engineering,Auxiliary financial activities,494.5952336
Civil engineering,Real estate activities,3697.318722
Civil engineering,Housing services,84401.19791
Civil engineering,Legal and accounting services,273.3548878
Civil engineering,Architectural and engineering services,1143.313193
Civil engineering,Other professional activities,1797.510836
Civil engineering,Rental and leasing activities,501.4236266
Civil engineering,Business support activities,1799.234703
Civil engineering,Public administration,13202.5123
Civil engineering,Public education,5299.434762
Civil engineering,Private education,2195.163925
Civil engineering,Public health activities,1769.916252
Civil engineering,Private health activities,1448.895143
Civil engineering,Activities of organizations,1387.744335
Civil engineering,Artistic and entertainment activities,630.7188259
Civil engineering,Other personal services,825.5264203
Specialized construction activities,Cultivation of annual crops,775.1944016
Specialized construction activities,Cultivation of vegetables,510.0736246
Specialized construction activities,Cultivation of grapes,1323.09791
Specialized construction activities,Cultivation of other fruits,4656.842099
Specialized construction activities,Cattle bredding,2683.703476
Specialized construction activities,Pigs breeding,1517.693402
Specialized construction activities,Chicken breeding,3117.869937
Specialized construction activities,Breeding of other animals,351.7735316
Specialized construction activities,Farming services,0.683164926
Specialized construction activities,Forestry,3981.654039
Specialized construction activities,Aquaculture,0.174469008
Specialized construction activities,Extractive fishing,3472.098674
Specialized construction activities,Coal mining,4.266509757
Specialized construction activities,Crude oil and gas mining,984.6554055
Specialized construction activities,Copper mining,4230.763312
Specialized construction activities,Iron mining,816.94764
Specialized construction activities,Mining of other metals,2174.952469
Specialized construction activities,Other mining activities and mining services,3347.597407
Specialized construction activities,Processing and preserving of meat,934.2077925
Specialized construction activities,Processing and preserving of fishmeal and fish oil,0.987127455
Specialized construction activities,Processing and preserving of fish,822.4548628
Specialized construction activities,Processing and preserving of fruits and vegetables,1100.842172
Specialized construction activities,Manufacture of vegetable and animal oils,312.5657078
Specialized construction activities,Manufacture of dairy products,1662.572407
Specialized construction activities,Manufacture of grain mill products,624.8652994
Specialized construction activities,Manufacture of prepared animal feeds,1039.475818
Specialized construction activities,Manufacture of bakery products,748.5425023
Specialized construction activities,Manufacture of macaroni noodles and other related products,134.3314809
Specialized construction activities,Manufacture of other food products,1045.046085
Specialized construction activities,Distilling rectifying and blending of spirits,130.2084861
Specialized construction activities,Manufacture of wines,2481.396392
Specialized construction activities,Manufacture of beers,16.39290228
Specialized construction activities,Manufacture of soft drinks,801.574006
Specialized construction activities,Manufacture of tobacco products,34.64455135
Specialized construction activities,Manufacture of textile products,801.867434
Specialized construction activities,Manufacture of wearing apparel,417.4143086
Specialized construction activities,Manufacture of leather products,82.62259556
Specialized construction activities,Manufacture of footwear,307.4676219
Specialized construction activities,Sawmilling and planing of wood,768.6378678
Specialized construction activities,Manufacture of wood products,599.0460353
Specialized construction activities,Manufacture of pulp and paper,1278.226734
Specialized construction activities,Manufacture of containers of paper,425.8798473
Specialized construction activities,Manufacture of other paper products,1467.474816
Specialized construction activities,Printing,726.5389169
Specialized construction activities,Manufacture of refined petroleum products,1411.978583
Specialized construction activities,Manufacture of basic chemicals,49.94474569
Specialized construction activities,Manufacture of paints,222.9949916
Specialized construction activities,Manufacture of pharmaceutical products,1653.715317
Specialized construction activities,Manufacture of soap detergents and toilet products,959.5966033
Specialized construction activities,Manufacture of other chemical products,5.290232278
Specialized construction activities,Manufacture of rubber products,213.8946533
Specialized construction activities,Manufacture of plastic products,1730.76855
Specialized construction activities,Manufacture of glass and glass products,567.9116469
Specialized construction activities,Manufacture of cement,2288.825007
Specialized construction activities,Manufacture of concrete and concrete products,1950.569324
Specialized construction activities,Manufacture of basic iron and steel,2.647218541
Specialized construction activities,Manufacture of other basic metals,195.7422273
Specialized construction activities,Manufacture of fabricated metal products,1963.396368
Specialized construction activities,Manufacture of industrial and domestic machinery and equipment,7.4574922
Specialized construction activities,Manufacture of electric and electronic machinery and equipment,323.890147
Specialized construction activities,Manufacture of transport equipment,626.4466973
Specialized construction activities,Manufacture of furniture,929.7319032
Specialized construction activities,Repair and installation of machinery and other manufacturing,524.7619153
Specialized construction activities,Electric power generation,9451.825654
Specialized construction activities,Transmission of electric power,64.13760381
Specialized construction activities,Distribution of electric power,14017.79591
Specialized construction activities,Gas and steam manufacture and supply,2966.12252
Specialized construction activities,Water collection treatment and supply,55000.43139
Specialized construction activities,Waste collection and recycling activities,1810.171813
Specialized construction activities,Construction of residential buildings,1220902.726
Specialized construction activities,Construction of non-residential buildings,965830.0329
Specialized construction activities,Civil engineering,273054.3304
Specialized construction activities,Specialized construction activities,32.53573332
Specialized construction activities,Wholesale and retail trade of motor vehicles,15450.28864
Specialized construction activities,Wholesale trade,76440.07966
Specialized construction activities,Retail trade,80997.09148
Specialized construction activities,Accommodation,15022.12622
Specialized construction activities,Food and beverage service activities,12512.9256
Specialized construction activities,Transport via railways,8040.642112
Specialized construction activities,Other passenger land transport,8887.33737
Specialized construction activities,Freight transport by road,11923.11448
Specialized construction activities,Transport via pipeline,14.6457239
Specialized construction activities,Water transport,5.747231304
Specialized construction activities,Air transport,5607.424584
Specialized construction activities,Warehousing,8056.260908
Specialized construction activities,Support activities for land transport,55094.47799
Specialized construction activities,Other support activities for transport,5588.3459
Specialized construction activities,Postal and courier activities,381.0098562
Specialized construction activities,Wireless telecommunications,55.7594688
Specialized construction activities,Wired telecommunications,13671.28363
Specialized construction activities,Other telecommunications activities,33.25778332
Specialized construction activities,Information services activities,4501.753217
Specialized construction activities,Publishing activities,2803.483252
Specialized construction activities,Banking monetary intermediation,3949.824583
Specialized construction activities,Insurance activities,2694.977367
Specialized construction activities,Auxiliary financial activities,2724.425974
Specialized construction activities,Real estate activities,70139.84548
Specialized construction activities,Housing services,1642750.042
Specialized construction activities,Legal and accounting services,3309.788262
Specialized construction activities,Architectural and engineering services,14067.87673
Specialized construction activities,Other professional activities,11163.60349
Specialized construction activities,Rental and leasing activities,7014.842137
Specialized construction activities,Business support activities,20860.92086
Specialized construction activities,Public administration,238997.8662
Specialized construction activities,Public education,102469.4151
Specialized construction activities,Private education,40950.52683
Specialized construction activities,Public health activities,28765.21318
Specialized construction activities,Private health activities,27078.89281
Specialized construction activities,Activities of organizations,26788.08184
Specialized construction activities,Artistic and entertainment activities,10065.9038
Specialized construction activities,Other personal services,15657.71951
Wholesale and retail trade of motor vehicles,Cultivation of annual crops,7394.504213
Wholesale and retail trade of motor vehicles,Cultivation of vegetables,2870.233798
Wholesale and retail trade of motor vehicles,Cultivation of grapes,2526.97
Wholesale and retail trade of motor vehicles,Cultivation of other fruits,8439.967193
Wholesale and retail trade of motor vehicles,Cattle bredding,3540.94739
Wholesale and retail trade of motor vehicles,Pigs breeding,466.6812107
Wholesale and retail trade of motor vehicles,Chicken breeding,538.6575365
Wholesale and retail trade of motor vehicles,Breeding of other animals,587.6491763
Wholesale and retail trade of motor vehicles,Farming services,4260.19293
Wholesale and retail trade of motor vehicles,Forestry,3869.885668
Wholesale and retail trade of motor vehicles,Aquaculture,2192.274321
Wholesale and retail trade of motor vehicles,Extractive fishing,7037.617322
Wholesale and retail trade of motor vehicles,Coal mining,181.0956581
Wholesale and retail trade of motor vehicles,Crude oil and gas mining,122.482341
Wholesale and retail trade of motor vehicles,Copper mining,64940.74104
Wholesale and retail trade of motor vehicles,Iron mining,859.6615168
Wholesale and retail trade of motor vehicles,Mining of other metals,2399.194167
Wholesale and retail trade of motor vehicles,Other mining activities and mining services,1928.572741
Wholesale and retail trade of motor vehicles,Processing and preserving of meat,4585.74998
Wholesale and retail trade of motor vehicles,Processing and preserving of fishmeal and fish oil,1383.00349
Wholesale and retail trade of motor vehicles,Processing and preserving of fish,2924.107311
Wholesale and retail trade of motor vehicles,Processing and preserving of fruits and vegetables,3658.286962
Wholesale and retail trade of motor vehicles,Manufacture of vegetable and animal oils,557.2364069
Wholesale and retail trade of motor vehicles,Manufacture of dairy products,2441.525375
Wholesale and retail trade of motor vehicles,Manufacture of grain mill products,489.1944493
Wholesale and retail trade of motor vehicles,Manufacture of prepared animal feeds,3088.495747
Wholesale and retail trade of motor vehicles,Manufacture of bakery products,12811.79118
Wholesale and retail trade of motor vehicles,Manufacture of macaroni noodles and other related products,161.4703923
Wholesale and retail trade of motor vehicles,Manufacture of other food products,1738.646318
Wholesale and retail trade of motor vehicles,Distilling rectifying and blending of spirits,130.8179249
Wholesale and retail trade of motor vehicles,Manufacture of wines,3298.755457
Wholesale and retail trade of motor vehicles,Manufacture of beers,777.5050903
Wholesale and retail trade of motor vehicles,Manufacture of soft drinks,1559.415889
Wholesale and retail trade of motor vehicles,Manufacture of tobacco products,204.9546228
Wholesale and retail trade of motor vehicles,Manufacture of textile products,2066.697532
Wholesale and retail trade of motor vehicles,Manufacture of wearing apparel,2255.97788
Wholesale and retail trade of motor vehicles,Manufacture of leather products,76.57593576
Wholesale and retail trade of motor vehicles,Manufacture of footwear,229.3105148
Wholesale and retail trade of motor vehicles,Sawmilling and planing of wood,2252.407584
Wholesale and retail trade of motor vehicles,Manufacture of wood products,951.594579
Wholesale and retail trade of motor vehicles,Manufacture of pulp and paper,4960.359932
Wholesale and retail trade of motor vehicles,Manufacture of containers of paper,757.7845929
Wholesale and retail trade of motor vehicles,Manufacture of other paper products,1486.924037
Wholesale and retail trade of motor vehicles,Printing,615.9939783
Wholesale and retail trade of motor vehicles,Manufacture of refined petroleum products,506.3409052
Wholesale and retail trade of motor vehicles,Manufacture of basic chemicals,2396.020478
Wholesale and retail trade of motor vehicles,Manufacture of paints,601.2012627
Wholesale and retail trade of motor vehicles,Manufacture of pharmaceutical products,1165.711752
Wholesale and retail trade of motor vehicles,Manufacture of soap detergents and toilet products,1248.218515
Wholesale and retail trade of motor vehicles,Manufacture of other chemical products,935.0870309
Wholesale and retail trade of motor vehicles,Manufacture of rubber products,1553.471049
Wholesale and retail trade of motor vehicles,Manufacture of plastic products,7249.577448
Wholesale and retail trade of motor vehicles,Manufacture of glass and glass products,1688.935535
Wholesale and retail trade of motor vehicles,Manufacture of cement,2838.223867
Wholesale and retail trade of motor vehicles,Manufacture of concrete and concrete products,6573.47276
Wholesale and retail trade of motor vehicles,Manufacture of basic iron and steel,3742.156721
Wholesale and retail trade of motor vehicles,Manufacture of other basic metals,1433.248328
Wholesale and retail trade of motor vehicles,Manufacture of fabricated metal products,10197.20836
Wholesale and retail trade of motor vehicles,Manufacture of industrial and domestic machinery and equipment,7198.429341
Wholesale and retail trade of motor vehicles,Manufacture of electric and electronic machinery and equipment,1353.563223
Wholesale and retail trade of motor vehicles,Manufacture of transport equipment,27596.08158
Wholesale and retail trade of motor vehicles,Manufacture of furniture,3960.888252
Wholesale and retail trade of motor vehicles,Repair and installation of machinery and other manufacturing,1683.04225
Wholesale and retail trade of motor vehicles,Electric power generation,5646.834919
Wholesale and retail trade of motor vehicles,Transmission of electric power,20.14074749
Wholesale and retail trade of motor vehicles,Distribution of electric power,1511.346732
Wholesale and retail trade of motor vehicles,Gas and steam manufacture and supply,258.1404977
Wholesale and retail trade of motor vehicles,Water collection treatment and supply,1119.067279
Wholesale and retail trade of motor vehicles,Waste collection and recycling activities,10405.09098
Wholesale and retail trade of motor vehicles,Construction of residential buildings,10589.08836
Wholesale and retail trade of motor vehicles,Construction of non-residential buildings,6719.613497
Wholesale and retail trade of motor vehicles,Civil engineering,33829.16006
Wholesale and retail trade of motor vehicles,Specialized construction activities,16469.74516
Wholesale and retail trade of motor vehicles,Wholesale and retail trade of motor vehicles,284252.4345
Wholesale and retail trade of motor vehicles,Wholesale trade,95569.48395
Wholesale and retail trade of motor vehicles,Retail trade,60879.70224
Wholesale and retail trade of motor vehicles,Accommodation,5770.211721
Wholesale and retail trade of motor vehicles,Food and beverage service activities,12217.90248
Wholesale and retail trade of motor vehicles,Transport via railways,235.7610415
Wholesale and retail trade of motor vehicles,Other passenger land transport,143147.9207
Wholesale and retail trade of motor vehicles,Freight transport by road,273847.4851
Wholesale and retail trade of motor vehicles,Transport via pipeline,10.12932818
Wholesale and retail trade of motor vehicles,Water transport,3694.494127
Wholesale and retail trade of motor vehicles,Air transport,20669.61541
Wholesale and retail trade of motor vehicles,Warehousing,3656.644137
Wholesale and retail trade of motor vehicles,Support activities for land transport,368.2632845
Wholesale and retail trade of motor vehicles,Other support activities for transport,7664.290659
Wholesale and retail trade of motor vehicles,Postal and courier activities,1476.922499
Wholesale and retail trade of motor vehicles,Wireless telecommunications,10923.2198
Wholesale and retail trade of motor vehicles,Wired telecommunications,3324.720954
Wholesale and retail trade of motor vehicles,Other telecommunications activities,3552.177421
Wholesale and retail trade of motor vehicles,Information services activities,5088.672674
Wholesale and retail trade of motor vehicles,Publishing activities,2165.246071
Wholesale and retail trade of motor vehicles,Banking monetary intermediation,3478.94335
Wholesale and retail trade of motor vehicles,Insurance activities,1844.241206
Wholesale and retail trade of motor vehicles,Auxiliary financial activities,1784.95414
Wholesale and retail trade of motor vehicles,Real estate activities,2982.463714
Wholesale and retail trade of motor vehicles,Housing services,0
Wholesale and retail trade of motor vehicles,Legal and accounting services,1878.268282
Wholesale and retail trade of motor vehicles,Architectural and engineering services,14235.48096
Wholesale and retail trade of motor vehicles,Other professional activities,6570.206147
Wholesale and retail trade of motor vehicles,Rental and leasing activities,111012.9867
Wholesale and retail trade of motor vehicles,Business support activities,26613.60239
Wholesale and retail trade of motor vehicles,Public administration,18210.91683
Wholesale and retail trade of motor vehicles,Public education,4001.867565
Wholesale and retail trade of motor vehicles,Private education,3113.13746
Wholesale and retail trade of motor vehicles,Public health activities,20243.14421
Wholesale and retail trade of motor vehicles,Private health activities,28706.95509
Wholesale and retail trade of motor vehicles,Activities of organizations,2144.84243
Wholesale and retail trade of motor vehicles,Artistic and entertainment activities,3475.426309
Wholesale and retail trade of motor vehicles,Other personal services,6069.514216
Wholesale trade,Cultivation of annual crops,78233.58168
Wholesale trade,Cultivation of vegetables,19620.81953
Wholesale trade,Cultivation of grapes,43164.60682
Wholesale trade,Cultivation of other fruits,103984.5783
Wholesale trade,Cattle bredding,50714.59366
Wholesale trade,Pigs breeding,13251.64338
Wholesale trade,Chicken breeding,20744.28644
Wholesale trade,Breeding of other animals,5171.216392
Wholesale trade,Farming services,28742.86198
Wholesale trade,Forestry,24864.99803
Wholesale trade,Aquaculture,58160.81463
Wholesale trade,Extractive fishing,17015.1569
Wholesale trade,Coal mining,2656.616022
Wholesale trade,Crude oil and gas mining,2091.000307
Wholesale trade,Copper mining,420509.928
Wholesale trade,Iron mining,27850.97922
Wholesale trade,Mining of other metals,32193.73831
Wholesale trade,Other mining activities and mining services,20839.04733
Wholesale trade,Processing and preserving of meat,44128.62186
Wholesale trade,Processing and preserving of fishmeal and fish oil,4960.732271
Wholesale trade,Processing and preserving of fish,17682.01224
Wholesale trade,Processing and preserving of fruits and vegetables,25945.15492
Wholesale trade,Manufacture of vegetable and animal oils,10985.14728
Wholesale trade,Manufacture of dairy products,39236.76592
Wholesale trade,Manufacture of grain mill products,10958.9464
Wholesale trade,Manufacture of prepared animal feeds,61593.79223
Wholesale trade,Manufacture of bakery products,38859.52279
Wholesale trade,Manufacture of macaroni noodles and other related products,3237.983446
Wholesale trade,Manufacture of other food products,36964.44175
Wholesale trade,Distilling rectifying and blending of spirits,2481.361093
Wholesale trade,Manufacture of wines,26670.87054
Wholesale trade,Manufacture of beers,11298.27862
Wholesale trade,Manufacture of soft drinks,30270.25743
Wholesale trade,Manufacture of tobacco products,1565.352778
Wholesale trade,Manufacture of textile products,17452.5868
Wholesale trade,Manufacture of wearing apparel,49482.33041
Wholesale trade,Manufacture of leather products,1745.580623
Wholesale trade,Manufacture of footwear,5142.140638
Wholesale trade,Sawmilling and planing of wood,4173.787231
Wholesale trade,Manufacture of wood products,23620.43076
Wholesale trade,Manufacture of pulp and paper,50764.16945
Wholesale trade,Manufacture of containers of paper,27864.73462
Wholesale trade,Manufacture of other paper products,35375.43894
Wholesale trade,Printing,22600.17927
Wholesale trade,Manufacture of refined petroleum products,7547.765422
Wholesale trade,Manufacture of basic chemicals,27788.30565
Wholesale trade,Manufacture of paints,10964.24129
Wholesale trade,Manufacture of pharmaceutical products,23245.38121
Wholesale trade,Manufacture of soap detergents and toilet products,26750.92296
Wholesale trade,Manufacture of other chemical products,19363.50826
Wholesale trade,Manufacture of rubber products,9813.350737
Wholesale trade,Manufacture of plastic products,80374.15932
Wholesale trade,Manufacture of glass and glass products,21318.50419
Wholesale trade,Manufacture of cement,20048.43865
Wholesale trade,Manufacture of concrete and concrete products,47217.23373
Wholesale trade,Manufacture of basic iron and steel,88662.79477
Wholesale trade,Manufacture of other basic metals,43880.15997
Wholesale trade,Manufacture of fabricated metal products,77285.90406
Wholesale trade,Manufacture of industrial and domestic machinery and equipment,81979.8371
Wholesale trade,Manufacture of electric and electronic machinery and equipment,32135.33737
Wholesale trade,Manufacture of transport equipment,17371.10107
Wholesale trade,Manufacture of furniture,49930.26891
Wholesale trade,Repair and installation of machinery and other manufacturing,28236.44045
Wholesale trade,Electric power generation,90684.95246
Wholesale trade,Transmission of electric power,7966.109905
Wholesale trade,Distribution of electric power,29616.41464
Wholesale trade,Gas and steam manufacture and supply,5787.990967
Wholesale trade,Water collection treatment and supply,4078.132047
Wholesale trade,Waste collection and recycling activities,58104.17287
Wholesale trade,Construction of residential buildings,150000.118
Wholesale trade,Construction of non-residential buildings,105184.2205
Wholesale trade,Civil engineering,286698.5467
Wholesale trade,Specialized construction activities,279234.6051
Wholesale trade,Wholesale and retail trade of motor vehicles,128853.5634
Wholesale trade,Wholesale trade,420490.6642
Wholesale trade,Retail trade,677926.7569
Wholesale trade,Accommodation,29385.35388
Wholesale trade,Food and beverage service activities,146057.5753
Wholesale trade,Transport via railways,10414.62963
Wholesale trade,Other passenger land transport,108733.8898
Wholesale trade,Freight transport by road,168051.0028
Wholesale trade,Transport via pipeline,4427.725707
Wholesale trade,Water transport,14650.04262
Wholesale trade,Air transport,31741.67152
Wholesale trade,Warehousing,1965.392636
Wholesale trade,Support activities for land transport,3227.335676
Wholesale trade,Other support activities for transport,49057.50882
Wholesale trade,Postal and courier activities,1759.92342
Wholesale trade,Wireless telecommunications,212294.9146
Wholesale trade,Wired telecommunications,80153.86683
Wholesale trade,Other telecommunications activities,107419.8278
Wholesale trade,Information services activities,34400.36355
Wholesale trade,Publishing activities,27781.27143
Wholesale trade,Banking monetary intermediation,32707.71004
Wholesale trade,Insurance activities,11171.55205
Wholesale trade,Auxiliary financial activities,18444.5909
Wholesale trade,Real estate activities,33073.74488
Wholesale trade,Housing services,0
Wholesale trade,Legal and accounting services,5409.074602
Wholesale trade,Architectural and engineering services,52244.32883
Wholesale trade,Other professional activities,48057.28444
Wholesale trade,Rental and leasing activities,91753.82832
Wholesale trade,Business support activities,144458.7516
Wholesale trade,Public administration,71542.83385
Wholesale trade,Public education,22053.49597
Wholesale trade,Private education,27002.52545
Wholesale trade,Public health activities,212943.8626
Wholesale trade,Private health activities,220026.2954
Wholesale trade,Activities of organizations,25305.4319
Wholesale trade,Artistic and entertainment activities,29784.05005
Wholesale trade,Other personal services,35570.54648
Retail trade,Cultivation of annual crops,10702.08419
Retail trade,Cultivation of vegetables,4548.118175
Retail trade,Cultivation of grapes,5403.179191
Retail trade,Cultivation of other fruits,14691.39154
Retail trade,Cattle bredding,6401.061151
Retail trade,Pigs breeding,1878.791255
Retail trade,Chicken breeding,1966.995096
Retail trade,Breeding of other animals,948.3096519
Retail trade,Farming services,4590.458734
Retail trade,Forestry,15463.88901
Retail trade,Aquaculture,9283.265777
Retail trade,Extractive fishing,7171.067529
Retail trade,Coal mining,535.4155293
Retail trade,Crude oil and gas mining,1013.629227
Retail trade,Copper mining,88440.17925
Retail trade,Iron mining,2865.048497
Retail trade,Mining of other metals,3948.557011
Retail trade,Other mining activities and mining services,4059.33096
Retail trade,Processing and preserving of meat,26230.93253
Retail trade,Processing and preserving of fishmeal and fish oil,1395.93914
Retail trade,Processing and preserving of fish,7754.738148
Retail trade,Processing and preserving of fruits and vegetables,16085.9475
Retail trade,Manufacture of vegetable and animal oils,4337.134852
Retail trade,Manufacture of dairy products,14972.44353
Retail trade,Manufacture of grain mill products,5133.43096
Retail trade,Manufacture of prepared animal feeds,27640.86256
Retail trade,Manufacture of bakery products,28741.39771
Retail trade,Manufacture of macaroni noodles and other related products,1222.082575
Retail trade,Manufacture of other food products,19881.41088
Retail trade,Distilling rectifying and blending of spirits,1721.224744
Retail trade,Manufacture of wines,8695.869152
Retail trade,Manufacture of beers,6684.995943
Retail trade,Manufacture of soft drinks,7710.946439
Retail trade,Manufacture of tobacco products,1442.435584
Retail trade,Manufacture of textile products,14826.69797
Retail trade,Manufacture of wearing apparel,45727.65832
Retail trade,Manufacture of leather products,757.9098582
Retail trade,Manufacture of footwear,4070.393503
Retail trade,Sawmilling and planing of wood,3797.643587
Retail trade,Manufacture of wood products,5244.440425
Retail trade,Manufacture of pulp and paper,8690.804038
Retail trade,Manufacture of containers of paper,16062.43674
Retail trade,Manufacture of other paper products,7333.060448
Retail trade,Printing,17284.59512
Retail trade,Manufacture of refined petroleum products,7192.916803
Retail trade,Manufacture of basic chemicals,16394.84493
Retail trade,Manufacture of paints,2927.702292
Retail trade,Manufacture of pharmaceutical products,9321.729859
Retail trade,Manufacture of soap detergents and toilet products,9467.535503
Retail trade,Manufacture of other chemical products,3607.326983
Retail trade,Manufacture of rubber products,2042.966235
Retail trade,Manufacture of plastic products,13491.63753
Retail trade,Manufacture of glass and glass products,2818.008373
Retail trade,Manufacture of cement,3315.767491
Retail trade,Manufacture of concrete and concrete products,11849.78309
Retail trade,Manufacture of basic iron and steel,7960.063783
Retail trade,Manufacture of other basic metals,2968.895109
Retail trade,Manufacture of fabricated metal products,15938.04206
Retail trade,Manufacture of industrial and domestic machinery and equipment,11639.09148
Retail trade,Manufacture of electric and electronic machinery and equipment,5194.1195
Retail trade,Manufacture of transport equipment,5256.939826
Retail trade,Manufacture of furniture,20052.3775
Retail trade,Repair and installation of machinery and other manufacturing,7757.081398
Retail trade,Electric power generation,8151.024313
Retail trade,Transmission of electric power,331.3769643
Retail trade,Distribution of electric power,2465.608988
Retail trade,Gas and steam manufacture and supply,5867.879747
Retail trade,Water collection treatment and supply,1564.155928
Retail trade,Waste collection and recycling activities,2649.951281
Retail trade,Construction of residential buildings,57726.67873
Retail trade,Construction of non-residential buildings,38376.84835
Retail trade,Civil engineering,138179.058
Retail trade,Specialized construction activities,117977.0977
Retail trade,Wholesale and retail trade of motor vehicles,26783.14408
Retail trade,Wholesale trade,110760.23
Retail trade,Retail trade,116708.5106
Retail trade,Accommodation,28550.42199
Retail trade,Food and beverage service activities,301789.3908
Retail trade,Transport via railways,750.8275527
Retail trade,Other passenger land transport,27478.93927
Retail trade,Freight transport by road,61075.86998
Retail trade,Transport via pipeline,92.10384568
Retail trade,Water transport,5257.645016
Retail trade,Air transport,13398.41713
Retail trade,Warehousing,2163.376296
Retail trade,Support activities for land transport,2728.450407
Retail trade,Other support activities for transport,11335.18238
Retail trade,Postal and courier activities,460.238922
Retail trade,Wireless telecommunications,27219.47801
Retail trade,Wired telecommunications,7897.246803
Retail trade,Other telecommunications activities,18360.29475
Retail trade,Information services activities,23411.28819
Retail trade,Publishing activities,12686.41244
Retail trade,Banking monetary intermediation,32666.01762
Retail trade,Insurance activities,18432.7634
Retail trade,Auxiliary financial activities,10723.12547
Retail trade,Real estate activities,15599.4197
Retail trade,Housing services,0
Retail trade,Legal and accounting services,11660.77585
Retail trade,Architectural and engineering services,24650.58442
Retail trade,Other professional activities,33613.70969
Retail trade,Rental and leasing activities,23033.07639
Retail trade,Business support activities,37228.08108
Retail trade,Public administration,57256.89574
Retail trade,Public education,15282.03559
Retail trade,Private education,15268.49085
Retail trade,Public health activities,43836.04458
Retail trade,Private health activities,38054.97076
Retail trade,Activities of organizations,14097.74866
Retail trade,Artistic and entertainment activities,12261.44352
Retail trade,Other personal services,7442.28683
Accommodation,Cultivation of annual crops,898.4017417
Accommodation,Cultivation of vegetables,535.3234812
Accommodation,Cultivation of grapes,1752.415693
Accommodation,Cultivation of other fruits,5442.167767
Accommodation,Cattle bredding,264.8837643
Accommodation,Pigs breeding,114.6521933
Accommodation,Chicken breeding,217.1415792
Accommodation,Breeding of other animals,28.00926738
Accommodation,Farming services,13.24639746
Accommodation,Forestry,1305.610946
Accommodation,Aquaculture,317.9424245
Accommodation,Extractive fishing,14.36448072
Accommodation,Coal mining,81.82726529
Accommodation,Crude oil and gas mining,1763.294736
Accommodation,Copper mining,13760.12896
Accommodation,Iron mining,376.8005694
Accommodation,Mining of other metals,2564.928236
Accommodation,Other mining activities and mining services,1190.263875
Accommodation,Processing and preserving of meat,741.7208268
Accommodation,Processing and preserving of fishmeal and fish oil,59.68067961
Accommodation,Processing and preserving of fish,611.5252812
Accommodation,Processing and preserving of fruits and vegetables,1292.55693
Accommodation,Manufacture of vegetable and animal oils,232.0337157
Accommodation,Manufacture of dairy products,1263.943341
Accommodation,Manufacture of grain mill products,671.7894362
Accommodation,Manufacture of prepared animal feeds,1407.834872
Accommodation,Manufacture of bakery products,1508.079995
Accommodation,Manufacture of macaroni noodles and other related products,61.06795697
Accommodation,Manufacture of other food products,1828.133834
Accommodation,Distilling rectifying and blending of spirits,386.3460818
Accommodation,Manufacture of wines,3057.370168
Accommodation,Manufacture of beers,454.99168
Accommodation,Manufacture of soft drinks,1295.506095
Accommodation,Manufacture of tobacco products,151.739831
Accommodation,Manufacture of textile products,653.2632424
Accommodation,Manufacture of wearing apparel,1624.795078
Accommodation,Manufacture of leather products,66.57750905
Accommodation,Manufacture of footwear,361.9496746
Accommodation,Sawmilling and planing of wood,643.8833276
Accommodation,Manufacture of wood products,879.3579581
Accommodation,Manufacture of pulp and paper,849.2919193
Accommodation,Manufacture of containers of paper,444.4326098
Accommodation,Manufacture of other paper products,573.1793458
Accommodation,Printing,1023.761573
Accommodation,Manufacture of refined petroleum products,718.7210124
Accommodation,Manufacture of basic chemicals,147.053954
Accommodation,Manufacture of paints,457.7073695
Accommodation,Manufacture of pharmaceutical products,3399.833311
Accommodation,Manufacture of soap detergents and toilet products,3418.90383
Accommodation,Manufacture of other chemical products,98.26560651
Accommodation,Manufacture of rubber products,802.0742643
Accommodation,Manufacture of plastic products,2640.223085
Accommodation,Manufacture of glass and glass products,356.6473987
Accommodation,Manufacture of cement,182.7710659
Accommodation,Manufacture of concrete and concrete products,1991.998545
Accommodation,Manufacture of basic iron and steel,69.76333525
Accommodation,Manufacture of other basic metals,409.2642783
Accommodation,Manufacture of fabricated metal products,2535.24991
Accommodation,Manufacture of industrial and domestic machinery and equipment,3159.027563
Accommodation,Manufacture of electric and electronic machinery and equipment,1845.903452
Accommodation,Manufacture of transport equipment,529.865087
Accommodation,Manufacture of furniture,953.7175697
Accommodation,Repair and installation of machinery and other manufacturing,701.1517992
Accommodation,Electric power generation,1645.619478
Accommodation,Transmission of electric power,235.3378852
Accommodation,Distribution of electric power,250.6311325
Accommodation,Gas and steam manufacture and supply,83.87291155
Accommodation,Water collection treatment and supply,551.5479385
Accommodation,Waste collection and recycling activities,459.7416162
Accommodation,Construction of residential buildings,1218.279767
Accommodation,Construction of non-residential buildings,621.147767
Accommodation,Civil engineering,6825.774897
Accommodation,Specialized construction activities,1069.092984
Accommodation,Wholesale and retail trade of motor vehicles,7248.516993
Accommodation,Wholesale trade,35894.39386
Accommodation,Retail trade,28317.92426
Accommodation,Accommodation,1838.16773
Accommodation,Food and beverage service activities,4870.758493
Accommodation,Transport via railways,18.31860013
Accommodation,Other passenger land transport,3557.116925
Accommodation,Freight transport by road,36575.2625
Accommodation,Transport via pipeline,65.75516013
Accommodation,Water transport,911.1556372
Accommodation,Air transport,6127.966246
Accommodation,Warehousing,968.8381777
Accommodation,Support activities for land transport,1270.358667
Accommodation,Other support activities for transport,2274.6864
Accommodation,Postal and courier activities,62.65089625
Accommodation,Wireless telecommunications,1942.289325
Accommodation,Wired telecommunications,438.2485763
Accommodation,Other telecommunications activities,2343.552148
Accommodation,Information services activities,13005.23138
Accommodation,Publishing activities,18937.47804
Accommodation,Banking monetary intermediation,11199.22224
Accommodation,Insurance activities,1737.605983
Accommodation,Auxiliary financial activities,4773.612181
Accommodation,Real estate activities,5063.361372
Accommodation,Housing services,0
Accommodation,Legal and accounting services,1298.617673
Accommodation,Architectural and engineering services,17812.58557
Accommodation,Other professional activities,32004.31179
Accommodation,Rental and leasing activities,5225.133751
Accommodation,Business support activities,12018.17339
Accommodation,Public administration,34941.44771
Accommodation,Public education,11357.02191
Accommodation,Private education,15625.51119
Accommodation,Public health activities,9564.80173
Accommodation,Private health activities,6278.898295
Accommodation,Activities of organizations,29955.68207
Accommodation,Artistic and entertainment activities,18332.16391
Accommodation,Other personal services,1132.511274
Food and beverage service activities,Cultivation of annual crops,1289.623438
Food and beverage service activities,Cultivation of vegetables,708.5991234
Food and beverage service activities,Cultivation of grapes,2463.435102
Food and beverage service activities,Cultivation of other fruits,7441.978423
Food and beverage service activities,Cattle bredding,309.187286
Food and beverage service activities,Pigs breeding,131.9955214
Food and beverage service activities,Chicken breeding,247.742692
Food and beverage service activities,Breeding of other animals,43.87648949
Food and beverage service activities,Farming services,83.04989498
Food and beverage service activities,Forestry,296.6221021
Food and beverage service activities,Aquaculture,153.4611805
Food and beverage service activities,Extractive fishing,24.63297195
Food and beverage service activities,Coal mining,2.671906941
Food and beverage service activities,Crude oil and gas mining,174.3872571
Food and beverage service activities,Copper mining,44229.03254
Food and beverage service activities,Iron mining,628.5529315
Food and beverage service activities,Mining of other metals,1126.568077
Food and beverage service activities,Other mining activities and mining services,36.99800219
Food and beverage service activities,Processing and preserving of meat,549.271991
Food and beverage service activities,Processing and preserving of fishmeal and fish oil,14.94425617
Food and beverage service activities,Processing and preserving of fish,120.433149
Food and beverage service activities,Processing and preserving of fruits and vegetables,278.7870384
Food and beverage service activities,Manufacture of vegetable and animal oils,77.16484717
Food and beverage service activities,Manufacture of dairy products,238.1643565
Food and beverage service activities,Manufacture of grain mill products,104.5684561
Food and beverage service activities,Manufacture of prepared animal feeds,302.9890068
Food and beverage service activities,Manufacture of bakery products,259.5747262
Food and beverage service activities,Manufacture of macaroni noodles and other related products,9.45441837
Food and beverage service activities,Manufacture of other food products,410.1864716
Food and beverage service activities,Distilling rectifying and blending of spirits,67.36663698
Food and beverage service activities,Manufacture of wines,512.7668419
Food and beverage service activities,Manufacture of beers,107.6421434
Food and beverage service activities,Manufacture of soft drinks,319.271277
Food and beverage service activities,Manufacture of tobacco products,113.0067543
Food and beverage service activities,Manufacture of textile products,173.6854194
Food and beverage service activities,Manufacture of wearing apparel,435.866418
Food and beverage service activities,Manufacture of leather products,20.27128534
Food and beverage service activities,Manufacture of footwear,110.3718095
Food and beverage service activities,Sawmilling and planing of wood,319.2862166
Food and beverage service activities,Manufacture of wood products,149.2318189
Food and beverage service activities,Manufacture of pulp and paper,261.7935738
Food and beverage service activities,Manufacture of containers of paper,832.162801
Food and beverage service activities,Manufacture of other paper products,198.7821493
Food and beverage service activities,Printing,240.0848404
Food and beverage service activities,Manufacture of refined petroleum products,233.3406023
Food and beverage service activities,Manufacture of basic chemicals,767.1765006
Food and beverage service activities,Manufacture of paints,98.99914638
Food and beverage service activities,Manufacture of pharmaceutical products,986.3164988
Food and beverage service activities,Manufacture of soap detergents and toilet products,462.8937711
Food and beverage service activities,Manufacture of other chemical products,317.6631405
Food and beverage service activities,Manufacture of rubber products,164.5681372
Food and beverage service activities,Manufacture of plastic products,53.88715425
Food and beverage service activities,Manufacture of glass and glass products,110.3173896
Food and beverage service activities,Manufacture of cement,196.0493236
Food and beverage service activities,Manufacture of concrete and concrete products,1368.650623
Food and beverage service activities,Manufacture of basic iron and steel,195.7959158
Food and beverage service activities,Manufacture of other basic metals,103.2350761
Food and beverage service activities,Manufacture of fabricated metal products,678.0241304
Food and beverage service activities,Manufacture of industrial and domestic machinery and equipment,591.8181324
Food and beverage service activities,Manufacture of electric and electronic machinery and equipment,437.4085725
Food and beverage service activities,Manufacture of transport equipment,133.3280974
Food and beverage service activities,Manufacture of furniture,286.9872986
Food and beverage service activities,Repair and installation of machinery and other manufacturing,111.4778587
Food and beverage service activities,Electric power generation,1894.8457
Food and beverage service activities,Transmission of electric power,3.023971787
Food and beverage service activities,Distribution of electric power,92.71218316
Food and beverage service activities,Gas and steam manufacture and supply,212.6092588
Food and beverage service activities,Water collection treatment and supply,429.4209367
Food and beverage service activities,Waste collection and recycling activities,591.3216727
Food and beverage service activities,Construction of residential buildings,1353.033905
Food and beverage service activities,Construction of non-residential buildings,1057.718369
Food and beverage service activities,Civil engineering,22235.94567
Food and beverage service activities,Specialized construction activities,4344.51831
Food and beverage service activities,Wholesale and retail trade of motor vehicles,3630.430722
Food and beverage service activities,Wholesale trade,18868.70746
Food and beverage service activities,Retail trade,6966.41006
Food and beverage service activities,Accommodation,100.2686176
Food and beverage service activities,Food and beverage service activities,951.5796498
Food and beverage service activities,Transport via railways,4.301574833
Food and beverage service activities,Other passenger land transport,9090.914871
Food and beverage service activities,Freight transport by road,69497.77046
Food and beverage service activities,Transport via pipeline,0.313188122
Food and beverage service activities,Water transport,4818.007902
Food and beverage service activities,Air transport,41123.86319
Food and beverage service activities,Warehousing,160.9459007
Food and beverage service activities,Support activities for land transport,34.19556719
Food and beverage service activities,Other support activities for transport,212.3854932
Food and beverage service activities,Postal and courier activities,5.456841841
Food and beverage service activities,Wireless telecommunications,1783.883398
Food and beverage service activities,Wired telecommunications,110.8220471
Food and beverage service activities,Other telecommunications activities,145.9093751
Food and beverage service activities,Information services activities,2770.486368
Food and beverage service activities,Publishing activities,1400.551579
Food and beverage service activities,Banking monetary intermediation,10115.74583
Food and beverage service activities,Insurance activities,6295.232308
Food and beverage service activities,Auxiliary financial activities,4001.984655
Food and beverage service activities,Real estate activities,872.4029307
Food and beverage service activities,Housing services,0
Food and beverage service activities,Legal and accounting services,1003.309882
Food and beverage service activities,Architectural and engineering services,7728.341201
Food and beverage service activities,Other professional activities,6043.255405
Food and beverage service activities,Rental and leasing activities,1611.896229
Food and beverage service activities,Business support activities,21255.53628
Food and beverage service activities,Public administration,27538.04358
Food and beverage service activities,Public education,95767.47863
Food and beverage service activities,Private education,25473.9253
Food and beverage service activities,Public health activities,8272.609668
Food and beverage service activities,Private health activities,22652.1401
Food and beverage service activities,Activities of organizations,15583.07416
Food and beverage service activities,Artistic and entertainment activities,8229.342369
Food and beverage service activities,Other personal services,14008.80637
Transport via railways,Cultivation of annual crops,16.44879242
Transport via railways,Cultivation of vegetables,7.560579958
Transport via railways,Cultivation of grapes,14.23351895
Transport via railways,Cultivation of other fruits,380.2361557
Transport via railways,Cattle bredding,14.74615321
Transport via railways,Pigs breeding,7.041027462
Transport via railways,Chicken breeding,12.44593429
Transport via railways,Breeding of other animals,1.578171748
Transport via railways,Farming services,0.422440895
Transport via railways,Forestry,119.2958692
Transport via railways,Aquaculture,130.0719188
Transport via railways,Extractive fishing,157.3865567
Transport via railways,Coal mining,1.3953997
Transport via railways,Crude oil and gas mining,12.67396453
Transport via railways,Copper mining,48242.94988
Transport via railways,Iron mining,2213.327298
Transport via railways,Mining of other metals,120.6648975
Transport via railways,Other mining activities and mining services,248.684547
Transport via railways,Processing and preserving of meat,118.3196797
Transport via railways,Processing and preserving of fishmeal and fish oil,61.84996249
Transport via railways,Processing and preserving of fish,376.1330409
Transport via railways,Processing and preserving of fruits and vegetables,1111.830436
Transport via railways,Manufacture of vegetable and animal oils,45.96153684
Transport via railways,Manufacture of dairy products,80.41156638
Transport via railways,Manufacture of grain mill products,147.3837872
Transport via railways,Manufacture of prepared animal feeds,4383.408022
Transport via railways,Manufacture of bakery products,172.8560485
Transport via railways,Manufacture of macaroni noodles and other related products,4.538361125
Transport via railways,Manufacture of other food products,99.36236435
Transport via railways,Distilling rectifying and blending of spirits,13.27762674
Transport via railways,Manufacture of wines,332.5803781
Transport via railways,Manufacture of beers,58.00932705
Transport via railways,Manufacture of soft drinks,189.6211435
Transport via railways,Manufacture of tobacco products,5.105073157
Transport via railways,Manufacture of textile products,525.7141928
Transport via railways,Manufacture of wearing apparel,72.81692843
Transport via railways,Manufacture of leather products,6.129327436
Transport via railways,Manufacture of footwear,24.16014118
Transport via railways,Sawmilling and planing of wood,1547.416983
Transport via railways,Manufacture of wood products,1827.345227
Transport via railways,Manufacture of pulp and paper,16567.13819
Transport via railways,Manufacture of containers of paper,172.3707555
Transport via railways,Manufacture of other paper products,147.6760653
Transport via railways,Printing,29.50732924
Transport via railways,Manufacture of refined petroleum products,96.16959996
Transport via railways,Manufacture of basic chemicals,221.3077526
Transport via railways,Manufacture of paints,12.3264237
Transport via railways,Manufacture of pharmaceutical products,136.9605702
Transport via railways,Manufacture of soap detergents and toilet products,367.7165938
Transport via railways,Manufacture of other chemical products,67.71734217
Transport via railways,Manufacture of rubber products,32.8147577
Transport via railways,Manufacture of plastic products,113.0448084
Transport via railways,Manufacture of glass and glass products,46.84693767
Transport via railways,Manufacture of cement,1991.841968
Transport via railways,Manufacture of concrete and concrete products,6594.862174
Transport via railways,Manufacture of basic iron and steel,111.6786715
Transport via railways,Manufacture of other basic metals,52.68717095
Transport via railways,Manufacture of fabricated metal products,305.6795478
Transport via railways,Manufacture of industrial and domestic machinery and equipment,258.2359393
Transport via railways,Manufacture of electric and electronic machinery and equipment,70.1761277
Transport via railways,Manufacture of transport equipment,137.699455
Transport via railways,Manufacture of furniture,197.562247
Transport via railways,Repair and installation of machinery and other manufacturing,51.49423164
Transport via railways,Electric power generation,252.6377206
Transport via railways,Transmission of electric power,17.07465295
Transport via railways,Distribution of electric power,83.11950137
Transport via railways,Gas and steam manufacture and supply,45.65956002
Transport via railways,Water collection treatment and supply,44.01019792
Transport via railways,Waste collection and recycling activities,126.715319
Transport via railways,Construction of residential buildings,792.9862771
Transport via railways,Construction of non-residential buildings,569.3070581
Transport via railways,Civil engineering,661.3557666
Transport via railways,Specialized construction activities,302.2277161
Transport via railways,Wholesale and retail trade of motor vehicles,623.6486023
Transport via railways,Wholesale trade,4637.794678
Transport via railways,Retail trade,2165.010347
Transport via railways,Accommodation,104.777219
Transport via railways,Food and beverage service activities,233.9783006
Transport via railways,Transport via railways,204.0474284
Transport via railways,Other passenger land transport,850.6798862
Transport via railways,Freight transport by road,1527.356325
Transport via railways,Transport via pipeline,8.923959275
Transport via railways,Water transport,402.3942149
Transport via railways,Air transport,1090.690226
Transport via railways,Warehousing,79.27089503
Transport via railways,Support activities for land transport,63.28848837
Transport via railways,Other support activities for transport,1173.050503
Transport via railways,Postal and courier activities,33.7099598
Transport via railways,Wireless telecommunications,191.0258674
Transport via railways,Wired telecommunications,112.9094595
Transport via railways,Other telecommunications activities,122.2303445
Transport via railways,Information services activities,178.1731998
Transport via railways,Publishing activities,82.78402972
Transport via railways,Banking monetary intermediation,146.5272317
Transport via railways,Insurance activities,38.89046364
Transport via railways,Auxiliary financial activities,34.26000917
Transport via railways,Real estate activities,257.7082687
Transport via railways,Housing services,835.0121861
Transport via railways,Legal and accounting services,102.7655219
Transport via railways,Architectural and engineering services,270.5553445
Transport via railways,Other professional activities,226.2272302
Transport via railways,Rental and leasing activities,281.4425188
Transport via railways,Business support activities,452.848356
Transport via railways,Public administration,477.9487018
Transport via railways,Public education,226.2065378
Transport via railways,Private education,133.8340112
Transport via railways,Public health activities,99.49893138
Transport via railways,Private health activities,199.549162
Transport via railways,Activities of organizations,52.77151831
Transport via railways,Artistic and entertainment activities,129.6434617
Transport via railways,Other personal services,66.10554915
Other passenger land transport,Cultivation of annual crops,3237.161826
Other passenger land transport,Cultivation of vegetables,1933.757553
Other passenger land transport,Cultivation of grapes,5701.486131
Other passenger land transport,Cultivation of other fruits,18551.97895
Other passenger land transport,Cattle bredding,993.4167585
Other passenger land transport,Pigs breeding,447.5300801
Other passenger land transport,Chicken breeding,897.5481329
Other passenger land transport,Breeding of other animals,138.522694
Other passenger land transport,Farming services,232.5844788
Other passenger land transport,Forestry,780.7987579
Other passenger land transport,Aquaculture,882.727962
Other passenger land transport,Extractive fishing,571.4736349
Other passenger land transport,Coal mining,605.8393733
Other passenger land transport,Crude oil and gas mining,1518.821767
Other passenger land transport,Copper mining,95411.18636
Other passenger land transport,Iron mining,2586.972085
Other passenger land transport,Mining of other metals,507.3777051
Other passenger land transport,Other mining activities and mining services,674.4590178
Other passenger land transport,Processing and preserving of meat,798.3959
Other passenger land transport,Processing and preserving of fishmeal and fish oil,251.2115792
Other passenger land transport,Processing and preserving of fish,1158.267936
Other passenger land transport,Processing and preserving of fruits and vegetables,1514.684904
Other passenger land transport,Manufacture of vegetable and animal oils,196.197843
Other passenger land transport,Manufacture of dairy products,654.8759635
Other passenger land transport,Manufacture of grain mill products,345.261748
Other passenger land transport,Manufacture of prepared animal feeds,4243.674377
Other passenger land transport,Manufacture of bakery products,508.4146449
Other passenger land transport,Manufacture of macaroni noodles and other related products,43.80163634
Other passenger land transport,Manufacture of other food products,599.5463264
Other passenger land transport,Distilling rectifying and blending of spirits,120.1340117
Other passenger land transport,Manufacture of wines,1472.74815
Other passenger land transport,Manufacture of beers,393.607254
Other passenger land transport,Manufacture of soft drinks,594.920726
Other passenger land transport,Manufacture of tobacco products,68.77147455
Other passenger land transport,Manufacture of textile products,716.1977497
Other passenger land transport,Manufacture of wearing apparel,551.7755508
Other passenger land transport,Manufacture of leather products,25.46455008
Other passenger land transport,Manufacture of footwear,87.98885871
Other passenger land transport,Sawmilling and planing of wood,2388.994481
Other passenger land transport,Manufacture of wood products,2435.784566
Other passenger land transport,Manufacture of pulp and paper,16427.50901
Other passenger land transport,Manufacture of containers of paper,384.0795996
Other passenger land transport,Manufacture of other paper products,613.4084118
Other passenger land transport,Printing,435.8621616
Other passenger land transport,Manufacture of refined petroleum products,467.7735263
Other passenger land transport,Manufacture of basic chemicals,1070.240916
Other passenger land transport,Manufacture of paints,99.64786482
Other passenger land transport,Manufacture of pharmaceutical products,931.9022118
Other passenger land transport,Manufacture of soap detergents and toilet products,781.0482837
Other passenger land transport,Manufacture of other chemical products,281.2359241
Other passenger land transport,Manufacture of rubber products,125.4296977
Other passenger land transport,Manufacture of plastic products,483.452619
Other passenger land transport,Manufacture of glass and glass products,189.3664097
Other passenger land transport,Manufacture of cement,2121.556093
Other passenger land transport,Manufacture of concrete and concrete products,6592.567062
Other passenger land transport,Manufacture of basic iron and steel,285.7159225
Other passenger land transport,Manufacture of other basic metals,148.3891075
Other passenger land transport,Manufacture of fabricated metal products,1168.884262
Other passenger land transport,Manufacture of industrial and domestic machinery and equipment,470.7607112
Other passenger land transport,Manufacture of electric and electronic machinery and equipment,253.3346832
Other passenger land transport,Manufacture of transport equipment,451.0681878
Other passenger land transport,Manufacture of furniture,544.1683622
Other passenger land transport,Repair and installation of machinery and other manufacturing,364.4733971
Other passenger land transport,Electric power generation,630.3986506
Other passenger land transport,Transmission of electric power,31.0162575
Other passenger land transport,Distribution of electric power,420.1729676
Other passenger land transport,Gas and steam manufacture and supply,442.7316867
Other passenger land transport,Water collection treatment and supply,382.5283655
Other passenger land transport,Waste collection and recycling activities,147.6404098
Other passenger land transport,Construction of residential buildings,1261.087333
Other passenger land transport,Construction of non-residential buildings,1859.221443
Other passenger land transport,Civil engineering,94737.95412
Other passenger land transport,Specialized construction activities,16281.91491
Other passenger land transport,Wholesale and retail trade of motor vehicles,2862.407645
Other passenger land transport,Wholesale trade,18877.553
Other passenger land transport,Retail trade,12844.37461
Other passenger land transport,Accommodation,782.9905992
Other passenger land transport,Food and beverage service activities,1309.394475
Other passenger land transport,Transport via railways,883.9482978
Other passenger land transport,Other passenger land transport,77999.95477
Other passenger land transport,Freight transport by road,7436.297119
Other passenger land transport,Transport via pipeline,10.33796295
Other passenger land transport,Water transport,1256.572778
Other passenger land transport,Air transport,8345.894256
Other passenger land transport,Warehousing,1422.158912
Other passenger land transport,Support activities for land transport,679.6830711
Other passenger land transport,Other support activities for transport,4563.825557
Other passenger land transport,Postal and courier activities,93.36842785
Other passenger land transport,Wireless telecommunications,2042.741974
Other passenger land transport,Wired telecommunications,959.4658903
Other passenger land transport,Other telecommunications activities,939.4351383
Other passenger land transport,Information services activities,2783.923191
Other passenger land transport,Publishing activities,755.7317596
Other passenger land transport,Banking monetary intermediation,15160.36327
Other passenger land transport,Insurance activities,5154.704596
Other passenger land transport,Auxiliary financial activities,2756.680181
Other passenger land transport,Real estate activities,1762.514376
Other passenger land transport,Housing services,0
Other passenger land transport,Legal and accounting services,1277.361403
Other passenger land transport,Architectural and engineering services,22492.43736
Other passenger land transport,Other professional activities,7983.362807
Other passenger land transport,Rental and leasing activities,1355.957795
Other passenger land transport,Business support activities,32317.94649
Other passenger land transport,Public administration,31328.89648
Other passenger land transport,Public education,17344.52128
Other passenger land transport,Private education,26858.13288
Other passenger land transport,Public health activities,39290.84091
Other passenger land transport,Private health activities,22215.33091
Other passenger land transport,Activities of organizations,14880.49335
Other passenger land transport,Artistic and entertainment activities,1064.580926
Other passenger land transport,Other personal services,267.0008303
Freight transport by road,Cultivation of annual crops,19644.27825
Freight transport by road,Cultivation of vegetables,8464.074983
Freight transport by road,Cultivation of grapes,17514.9896
Freight transport by road,Cultivation of other fruits,56486.10558
Freight transport by road,Cattle bredding,17163.27127
Freight transport by road,Pigs breeding,8359.392112
Freight transport by road,Chicken breeding,14367.14514
Freight transport by road,Breeding of other animals,1939.974038
Freight transport by road,Farming services,52.79661104
Freight transport by road,Forestry,26873.93034
Freight transport by road,Aquaculture,32283.28156
Freight transport by road,Extractive fishing,138.7223625
Freight transport by road,Coal mining,196.1088259
Freight transport by road,Crude oil and gas mining,705.2259076
Freight transport by road,Copper mining,179309.6844
Freight transport by road,Iron mining,2804.627307
Freight transport by road,Mining of other metals,7186.922831
Freight transport by road,Other mining activities and mining services,93162.76344
Freight transport by road,Processing and preserving of meat,26249.75515
Freight transport by road,Processing and preserving of fishmeal and fish oil,5823.120296
Freight transport by road,Processing and preserving of fish,16937.36717
Freight transport by road,Processing and preserving of fruits and vegetables,30323.74739
Freight transport by road,Manufacture of vegetable and animal oils,9829.313975
Freight transport by road,Manufacture of dairy products,46792.05879
Freight transport by road,Manufacture of grain mill products,22370.56536
Freight transport by road,Manufacture of prepared animal feeds,30149.18321
Freight transport by road,Manufacture of bakery products,26813.21306
Freight transport by road,Manufacture of macaroni noodles and other related products,1879.760797
Freight transport by road,Manufacture of other food products,43479.6494
Freight transport by road,Distilling rectifying and blending of spirits,10233.29766
Freight transport by road,Manufacture of wines,28689.16396
Freight transport by road,Manufacture of beers,54027.80014
Freight transport by road,Manufacture of soft drinks,171858.1652
Freight transport by road,Manufacture of tobacco products,1395.087228
Freight transport by road,Manufacture of textile products,3268.692767
Freight transport by road,Manufacture of wearing apparel,5429.555084
Freight transport by road,Manufacture of leather products,368.1340407
Freight transport by road,Manufacture of footwear,1790.705006
Freight transport by road,Sawmilling and planing of wood,89030.98631
Freight transport by road,Manufacture of wood products,95547.5682
Freight transport by road,Manufacture of pulp and paper,177837.1518
Freight transport by road,Manufacture of containers of paper,19764.03223
Freight transport by road,Manufacture of other paper products,25545.40473
Freight transport by road,Printing,10083.15237
Freight transport by road,Manufacture of refined petroleum products,74932.88871
Freight transport by road,Manufacture of basic chemicals,40429.44506
Freight transport by road,Manufacture of paints,8408.409124
Freight transport by road,Manufacture of pharmaceutical products,12940.18541
Freight transport by road,Manufacture of soap detergents and toilet products,23654.09889
Freight transport by road,Manufacture of other chemical products,17919.3374
Freight transport by road,Manufacture of rubber products,3653.193999
Freight transport by road,Manufacture of plastic products,36912.15495
Freight transport by road,Manufacture of glass and glass products,6124.023012
Freight transport by road,Manufacture of cement,62566.84304
Freight transport by road,Manufacture of concrete and concrete products,26237.57155
Freight transport by road,Manufacture of basic iron and steel,13077.23082
Freight transport by road,Manufacture of other basic metals,854.5494423
Freight transport by road,Manufacture of fabricated metal products,36023.76961
Freight transport by road,Manufacture of industrial and domestic machinery and equipment,46175.51381
Freight transport by road,Manufacture of electric and electronic machinery and equipment,7873.569575
Freight transport by road,Manufacture of transport equipment,2362.206279
Freight transport by road,Manufacture of furniture,23596.39329
Freight transport by road,Repair and installation of machinery and other manufacturing,10907.48603
Freight transport by road,Electric power generation,9124.861048
Freight transport by road,Transmission of electric power,80.52116169
Freight transport by road,Distribution of electric power,426.366916
Freight transport by road,Gas and steam manufacture and supply,186.0561689
Freight transport by road,Water collection treatment and supply,359.1147369
Freight transport by road,Waste collection and recycling activities,21988.90723
Freight transport by road,Construction of residential buildings,52235.50791
Freight transport by road,Construction of non-residential buildings,6063.61076
Freight transport by road,Civil engineering,54885.45745
Freight transport by road,Specialized construction activities,12724.44569
Freight transport by road,Wholesale and retail trade of motor vehicles,74027.98048
Freight transport by road,Wholesale trade,1212943.223
Freight transport by road,Retail trade,503787.6236
Freight transport by road,Accommodation,4599.391512
Freight transport by road,Food and beverage service activities,37378.53418
Freight transport by road,Transport via railways,4059.523053
Freight transport by road,Other passenger land transport,3375.897003
Freight transport by road,Freight transport by road,877897.7451
Freight transport by road,Transport via pipeline,4.038638023
Freight transport by road,Water transport,17296.83461
Freight transport by road,Air transport,2882.619471
Freight transport by road,Warehousing,30288.01874
Freight transport by road,Support activities for land transport,227.4087314
Freight transport by road,Other support activities for transport,14044.51895
Freight transport by road,Postal and courier activities,32820.59835
Freight transport by road,Wireless telecommunications,604.0246147
Freight transport by road,Wired telecommunications,546.4554162
Freight transport by road,Other telecommunications activities,2057.348284
Freight transport by road,Information services activities,5215.04213
Freight transport by road,Publishing activities,20851.88974
Freight transport by road,Banking monetary intermediation,5511.100315
Freight transport by road,Insurance activities,131.7482266
Freight transport by road,Auxiliary financial activities,415.474147
Freight transport by road,Real estate activities,4376.427683
Freight transport by road,Housing services,0
Freight transport by road,Legal and accounting services,1161.844618
Freight transport by road,Architectural and engineering services,37079.34197
Freight transport by road,Other professional activities,14272.92922
Freight transport by road,Rental and leasing activities,45988.95248
Freight transport by road,Business support activities,73843.91443
Freight transport by road,Public administration,15221.28754
Freight transport by road,Public education,7974.296033
Freight transport by road,Private education,2017.583344
Freight transport by road,Public health activities,12644.18362
Freight transport by road,Private health activities,11035.4683
Freight transport by road,Activities of organizations,8838.027655
Freight transport by road,Artistic and entertainment activities,7910.379938
Freight transport by road,Other personal services,7836.924066
Transport via pipeline,Cultivation of annual crops,0.307518574
Transport via pipeline,Cultivation of vegetables,0.115064666
Transport via pipeline,Cultivation of grapes,0.187988343
Transport via pipeline,Cultivation of other fruits,0.46056326
Transport via pipeline,Cattle bredding,0.357719275
Transport via pipeline,Pigs breeding,0.145187881
Transport via pipeline,Chicken breeding,0.254561989
Transport via pipeline,Breeding of other animals,0.037185073
Transport via pipeline,Farming services,0.344449389
Transport via pipeline,Forestry,0.205709443
Transport via pipeline,Aquaculture,3.845742365
Transport via pipeline,Extractive fishing,8.862854485
Transport via pipeline,Coal mining,0.157812942
Transport via pipeline,Crude oil and gas mining,3349.324953
Transport via pipeline,Copper mining,219.549324
Transport via pipeline,Iron mining,21.65231095
Transport via pipeline,Mining of other metals,15.23722511
Transport via pipeline,Other mining activities and mining services,15.21172853
Transport via pipeline,Processing and preserving of meat,11.79300997
Transport via pipeline,Processing and preserving of fishmeal and fish oil,3.656610071
Transport via pipeline,Processing and preserving of fish,14.81191388
Transport via pipeline,Processing and preserving of fruits and vegetables,9.232650541
Transport via pipeline,Manufacture of vegetable and animal oils,2.921201676
Transport via pipeline,Manufacture of dairy products,6.044554822
Transport via pipeline,Manufacture of grain mill products,4.049084593
Transport via pipeline,Manufacture of prepared animal feeds,3.326985179
Transport via pipeline,Manufacture of bakery products,3.484886311
Transport via pipeline,Manufacture of macaroni noodles and other related products,0.103716575
Transport via pipeline,Manufacture of other food products,7.22604626
Transport via pipeline,Distilling rectifying and blending of spirits,2.568522427
Transport via pipeline,Manufacture of wines,50.10426944
Transport via pipeline,Manufacture of beers,12.55338269
Transport via pipeline,Manufacture of soft drinks,35.16504039
Transport via pipeline,Manufacture of tobacco products,1.517730446
Transport via pipeline,Manufacture of textile products,3.429646568
Transport via pipeline,Manufacture of wearing apparel,6.280979201
Transport via pipeline,Manufacture of leather products,0.340315564
Transport via pipeline,Manufacture of footwear,0.667109802
Transport via pipeline,Sawmilling and planing of wood,19.02637307
Transport via pipeline,Manufacture of wood products,11.71813483
Transport via pipeline,Manufacture of pulp and paper,20.60416753
Transport via pipeline,Manufacture of containers of paper,3.962025118
Transport via pipeline,Manufacture of other paper products,4.834150616
Transport via pipeline,Printing,3.454780907
Transport via pipeline,Manufacture of refined petroleum products,5.694760575
Transport via pipeline,Manufacture of basic chemicals,15.6732187
Transport via pipeline,Manufacture of paints,1.298571027
Transport via pipeline,Manufacture of pharmaceutical products,15.67156851
Transport via pipeline,Manufacture of soap detergents and toilet products,6.570052676
Transport via pipeline,Manufacture of other chemical products,4.789330391
Transport via pipeline,Manufacture of rubber products,1.631001955
Transport via pipeline,Manufacture of plastic products,5.781525905
Transport via pipeline,Manufacture of glass and glass products,2.391610363
Transport via pipeline,Manufacture of cement,2.455051531
Transport via pipeline,Manufacture of concrete and concrete products,6.842612529
Transport via pipeline,Manufacture of basic iron and steel,4.023654449
Transport via pipeline,Manufacture of other basic metals,2.113495154
Transport via pipeline,Manufacture of fabricated metal products,12.12966218
Transport via pipeline,Manufacture of industrial and domestic machinery and equipment,6.296855723
Transport via pipeline,Manufacture of electric and electronic machinery and equipment,2.631365969
Transport via pipeline,Manufacture of transport equipment,2.340414037
Transport via pipeline,Manufacture of furniture,5.197865892
Transport via pipeline,Repair and installation of machinery and other manufacturing,1.595299438
Transport via pipeline,Electric power generation,76285.37691
Transport via pipeline,Transmission of electric power,0.156210451
Transport via pipeline,Distribution of electric power,0.708283878
Transport via pipeline,Gas and steam manufacture and supply,55723.72439
Transport via pipeline,Water collection treatment and supply,2.066710305
Transport via pipeline,Waste collection and recycling activities,0.613319712
Transport via pipeline,Construction of residential buildings,9.671259547
Transport via pipeline,Construction of non-residential buildings,4.132262019
Transport via pipeline,Civil engineering,34.25182669
Transport via pipeline,Specialized construction activities,15.31903314
Transport via pipeline,Wholesale and retail trade of motor vehicles,39.81004327
Transport via pipeline,Wholesale trade,399.7945232
Transport via pipeline,Retail trade,104.6381719
Transport via pipeline,Accommodation,1.182843575
Transport via pipeline,Food and beverage service activities,2.478200379
Transport via pipeline,Transport via railways,24.47668918
Transport via pipeline,Other passenger land transport,52.14320717
Transport via pipeline,Freight transport by road,48.09887281
Transport via pipeline,Transport via pipeline,16132.96197
Transport via pipeline,Water transport,56.25436583
Transport via pipeline,Air transport,63.80440449
Transport via pipeline,Warehousing,2.39980276
Transport via pipeline,Support activities for land transport,2.150120051
Transport via pipeline,Other support activities for transport,115.8075062
Transport via pipeline,Postal and courier activities,0.841786518
Transport via pipeline,Wireless telecommunications,5.939051613
Transport via pipeline,Wired telecommunications,4.094466784
Transport via pipeline,Other telecommunications activities,6.729242091
Transport via pipeline,Information services activities,16.65126753
Transport via pipeline,Publishing activities,4.361693253
Transport via pipeline,Banking monetary intermediation,20.2069102
Transport via pipeline,Insurance activities,10.59573223
Transport via pipeline,Auxiliary financial activities,4.717692491
Transport via pipeline,Real estate activities,11.02067988
Transport via pipeline,Housing services,0
Transport via pipeline,Legal and accounting services,5.752901849
Transport via pipeline,Architectural and engineering services,84.35394487
Transport via pipeline,Other professional activities,32.68175685
Transport via pipeline,Rental and leasing activities,5.810741848
Transport via pipeline,Business support activities,22.97235982
Transport via pipeline,Public administration,19.46913567
Transport via pipeline,Public education,3.738018169
Transport via pipeline,Private education,7.283131915
Transport via pipeline,Public health activities,3.848500426
Transport via pipeline,Private health activities,5.721590354
Transport via pipeline,Activities of organizations,2.671179896
Transport via pipeline,Artistic and entertainment activities,2.925879101
Transport via pipeline,Other personal services,1.237036815
Water transport,Cultivation of annual crops,3.346758536
Water transport,Cultivation of vegetables,1.653408902
Water transport,Cultivation of grapes,2.435184299
Water transport,Cultivation of other fruits,6.556455787
Water transport,Cattle bredding,3.048987331
Water transport,Pigs breeding,1.207899507
Water transport,Chicken breeding,2.115601168
Water transport,Breeding of other animals,0.265733423
Water transport,Farming services,0.37432427
Water transport,Forestry,16.76357535
Water transport,Aquaculture,131817.3827
Water transport,Extractive fishing,267.7339151
Water transport,Coal mining,8.327984496
Water transport,Crude oil and gas mining,274.7959266
Water transport,Copper mining,74479.21196
Water transport,Iron mining,301.1418193
Water transport,Mining of other metals,1159.525734
Water transport,Other mining activities and mining services,1053.304195
Water transport,Processing and preserving of meat,197.6110214
Water transport,Processing and preserving of fishmeal and fish oil,86.01137238
Water transport,Processing and preserving of fish,10139.26063
Water transport,Processing and preserving of fruits and vegetables,351.9924613
Water transport,Manufacture of vegetable and animal oils,117.230642
Water transport,Manufacture of dairy products,94.7374043
Water transport,Manufacture of grain mill products,47.33655715
Water transport,Manufacture of prepared animal feeds,16351.33745
Water transport,Manufacture of bakery products,288.8312565
Water transport,Manufacture of macaroni noodles and other related products,2.720825211
Water transport,Manufacture of other food products,125.4535711
Water transport,Distilling rectifying and blending of spirits,11.8787845
Water transport,Manufacture of wines,379.0163375
Water transport,Manufacture of beers,19.09547378
Water transport,Manufacture of soft drinks,67.11345349
Water transport,Manufacture of tobacco products,4.956711401
Water transport,Manufacture of textile products,134.5797173
Water transport,Manufacture of wearing apparel,91.15318235
Water transport,Manufacture of leather products,35.72828927
Water transport,Manufacture of footwear,12.11267718
Water transport,Sawmilling and planing of wood,396.3238825
Water transport,Manufacture of wood products,679.6132573
Water transport,Manufacture of pulp and paper,441.3070701
Water transport,Manufacture of containers of paper,76.61397031
Water transport,Manufacture of other paper products,63.81585596
Water transport,Printing,23.77093371
Water transport,Manufacture of refined petroleum products,51988.33579
Water transport,Manufacture of basic chemicals,27228.08942
Water transport,Manufacture of paints,13.5303911
Water transport,Manufacture of pharmaceutical products,210.9733117
Water transport,Manufacture of soap detergents and toilet products,106.6842026
Water transport,Manufacture of other chemical products,9163.711637
Water transport,Manufacture of rubber products,39.82970599
Water transport,Manufacture of plastic products,113.3142756
Water transport,Manufacture of glass and glass products,54.89625822
Water transport,Manufacture of cement,5607.631713
Water transport,Manufacture of concrete and concrete products,143.7751301
Water transport,Manufacture of basic iron and steel,1705.79313
Water transport,Manufacture of other basic metals,40.8881819
Water transport,Manufacture of fabricated metal products,3211.355507
Water transport,Manufacture of industrial and domestic machinery and equipment,111.0076109
Water transport,Manufacture of electric and electronic machinery and equipment,54.68145423
Water transport,Manufacture of transport equipment,1886.899655
Water transport,Manufacture of furniture,285.5638079
Water transport,Repair and installation of machinery and other manufacturing,97.51595605
Water transport,Electric power generation,107.4499676
Water transport,Transmission of electric power,1.496023484
Water transport,Distribution of electric power,15.77212225
Water transport,Gas and steam manufacture and supply,59.3934225
Water transport,Water collection treatment and supply,6.175254562
Water transport,Waste collection and recycling activities,8.130252695
Water transport,Construction of residential buildings,49.06992503
Water transport,Construction of non-residential buildings,21.82747806
Water transport,Civil engineering,74.37592256
Water transport,Specialized construction activities,7.527332561
Water transport,Wholesale and retail trade of motor vehicles,897.6543437
Water transport,Wholesale trade,8254.963066
Water transport,Retail trade,2159.117719
Water transport,Accommodation,62.40765232
Water transport,Food and beverage service activities,256.8658683
Water transport,Transport via railways,226.3514665
Water transport,Other passenger land transport,1526.307235
Water transport,Freight transport by road,8195.686002
Water transport,Transport via pipeline,0.478194668
Water transport,Water transport,19093.99963
Water transport,Air transport,6564.595178
Water transport,Warehousing,891.9022302
Water transport,Support activities for land transport,27.07145299
Water transport,Other support activities for transport,3677.09349
Water transport,Postal and courier activities,14.92749356
Water transport,Wireless telecommunications,118.9632577
Water transport,Wired telecommunications,57.60636813
Water transport,Other telecommunications activities,30.89336584
Water transport,Information services activities,446.9113969
Water transport,Publishing activities,48.85552511
Water transport,Banking monetary intermediation,261.0052081
Water transport,Insurance activities,72.51574132
Water transport,Auxiliary financial activities,51.04288002
Water transport,Real estate activities,116.609079
Water transport,Housing services,0
Water transport,Legal and accounting services,221.0833933
Water transport,Architectural and engineering services,309.422966
Water transport,Other professional activities,285.7044407
Water transport,Rental and leasing activities,87.97704748
Water transport,Business support activities,3922.774745
Water transport,Public administration,2829.231211
Water transport,Public education,83.7240353
Water transport,Private education,72.28481375
Water transport,Public health activities,46.92646467
Water transport,Private health activities,62.84329309
Water transport,Activities of organizations,13.87968099
Water transport,Artistic and entertainment activities,53.52562205
Water transport,Other personal services,283.585026
Air transport,Cultivation of annual crops,908.6862489
Air transport,Cultivation of vegetables,494.2139484
Air transport,Cultivation of grapes,857.7062254
Air transport,Cultivation of other fruits,2289.474034
Air transport,Cattle bredding,477.026046
Air transport,Pigs breeding,189.8147546
Air transport,Chicken breeding,394.7469842
Air transport,Breeding of other animals,45.09404649
Air transport,Farming services,131.3238954
Air transport,Forestry,19527.85874
Air transport,Aquaculture,18591.19203
Air transport,Extractive fishing,252.2950618
Air transport,Coal mining,363.6661198
Air transport,Crude oil and gas mining,3852.580759
Air transport,Copper mining,88238.48197
Air transport,Iron mining,4596.612098
Air transport,Mining of other metals,11732.90138
Air transport,Other mining activities and mining services,4090.598582
Air transport,Processing and preserving of meat,7444.970741
Air transport,Processing and preserving of fishmeal and fish oil,449.7890313
Air transport,Processing and preserving of fish,10333.41476
Air transport,Processing and preserving of fruits and vegetables,3782.443422
Air transport,Manufacture of vegetable and animal oils,617.8676869
Air transport,Manufacture of dairy products,4245.135112
Air transport,Manufacture of grain mill products,1675.72999
Air transport,Manufacture of prepared animal feeds,3526.724955
Air transport,Manufacture of bakery products,3258.657157
Air transport,Manufacture of macaroni noodles and other related products,99.18756017
Air transport,Manufacture of other food products,3872.57089
Air transport,Distilling rectifying and blending of spirits,757.6897724
Air transport,Manufacture of wines,9359.987298
Air transport,Manufacture of beers,1356.844731
Air transport,Manufacture of soft drinks,4288.319452
Air transport,Manufacture of tobacco products,647.7584266
Air transport,Manufacture of textile products,1325.444847
Air transport,Manufacture of wearing apparel,2777.688973
Air transport,Manufacture of leather products,144.6942823
Air transport,Manufacture of footwear,1006.072534
Air transport,Sawmilling and planing of wood,2074.4918
Air transport,Manufacture of wood products,2754.274847
Air transport,Manufacture of pulp and paper,2532.224167
Air transport,Manufacture of containers of paper,1613.331464
Air transport,Manufacture of other paper products,2397.124117
Air transport,Printing,3343.719428
Air transport,Manufacture of refined petroleum products,3073.198565
Air transport,Manufacture of basic chemicals,9455.206115
Air transport,Manufacture of paints,984.562402
Air transport,Manufacture of pharmaceutical products,5713.719852
Air transport,Manufacture of soap detergents and toilet products,6173.628766
Air transport,Manufacture of other chemical products,5722.695034
Air transport,Manufacture of rubber products,1748.593639
Air transport,Manufacture of plastic products,7270.3735
Air transport,Manufacture of glass and glass products,1174.230899
Air transport,Manufacture of cement,1431.412483
Air transport,Manufacture of concrete and concrete products,6396.031848
Air transport,Manufacture of basic iron and steel,4290.360542
Air transport,Manufacture of other basic metals,1579.640944
Air transport,Manufacture of fabricated metal products,8578.428117
Air transport,Manufacture of industrial and domestic machinery and equipment,7245.423801
Air transport,Manufacture of electric and electronic machinery and equipment,3540.79168
Air transport,Manufacture of transport equipment,1206.715274
Air transport,Manufacture of furniture,2382.32625
Air transport,Repair and installation of machinery and other manufacturing,3360.215159
Air transport,Electric power generation,15491.17612
Air transport,Transmission of electric power,668.6604605
Air transport,Distribution of electric power,4894.93067
Air transport,Gas and steam manufacture and supply,2034.75506
Air transport,Water collection treatment and supply,3210.543966
Air transport,Waste collection and recycling activities,6824.639934
Air transport,Construction of residential buildings,21945.6014
Air transport,Construction of non-residential buildings,15302.56196
Air transport,Civil engineering,92399.50539
Air transport,Specialized construction activities,8165.648685
Air transport,Wholesale and retail trade of motor vehicles,7420.116846
Air transport,Wholesale trade,36085.81991
Air transport,Retail trade,30048.0572
Air transport,Accommodation,3627.63996
Air transport,Food and beverage service activities,16013.38672
Air transport,Transport via railways,778.2314842
Air transport,Other passenger land transport,8685.471511
Air transport,Freight transport by road,59263.02873
Air transport,Transport via pipeline,338.2428981
Air transport,Water transport,5206.451587
Air transport,Air transport,22738.4629
Air transport,Warehousing,1325.606864
Air transport,Support activities for land transport,2792.93029
Air transport,Other support activities for transport,16103.75947
Air transport,Postal and courier activities,27103.07054
Air transport,Wireless telecommunications,14529.07113
Air transport,Wired telecommunications,9430.79268
Air transport,Other telecommunications activities,9449.709268
Air transport,Information services activities,23307.26522
Air transport,Publishing activities,10854.50047
Air transport,Banking monetary intermediation,17158.03614
Air transport,Insurance activities,10548.01707
Air transport,Auxiliary financial activities,7510.39058
Air transport,Real estate activities,5004.116025
Air transport,Housing services,0
Air transport,Legal and accounting services,33849.50145
Air transport,Architectural and engineering services,50743.43457
Air transport,Other professional activities,25925.54845
Air transport,Rental and leasing activities,24112.88475
Air transport,Business support activities,34516.88095
Air transport,Public administration,74794.07189
Air transport,Public education,5077.25294
Air transport,Private education,3188.518912
Air transport,Public health activities,6735.089325
Air transport,Private health activities,8715.642116
Air transport,Activities of organizations,1219.10269
Air transport,Artistic and entertainment activities,11665.03263
Air transport,Other personal services,1700.505742
Warehousing,Cultivation of annual crops,0.937165988
Warehousing,Cultivation of vegetables,0.710762952
Warehousing,Cultivation of grapes,1.328372603
Warehousing,Cultivation of other fruits,2.527327614
Warehousing,Cattle bredding,0.902118533
Warehousing,Pigs breeding,0.134284242
Warehousing,Chicken breeding,0.164649272
Warehousing,Breeding of other animals,0.063253817
Warehousing,Farming services,0.48665363
Warehousing,Forestry,1.229088423
Warehousing,Aquaculture,227.0301103
Warehousing,Extractive fishing,0.244238677
Warehousing,Coal mining,0
Warehousing,Crude oil and gas mining,102.7388991
Warehousing,Copper mining,324.6525502
Warehousing,Iron mining,0.09413443
Warehousing,Mining of other metals,0.537550263
Warehousing,Other mining activities and mining services,0.367171726
Warehousing,Processing and preserving of meat,10.3193677
Warehousing,Processing and preserving of fishmeal and fish oil,0.556864875
Warehousing,Processing and preserving of fish,2.375119541
Warehousing,Processing and preserving of fruits and vegetables,0.489162911
Warehousing,Manufacture of vegetable and animal oils,0.323762394
Warehousing,Manufacture of dairy products,59.97168256
Warehousing,Manufacture of grain mill products,22.41488238
Warehousing,Manufacture of prepared animal feeds,12.64931704
Warehousing,Manufacture of bakery products,39.59098089
Warehousing,Manufacture of macaroni noodles and other related products,0.58354387
Warehousing,Manufacture of other food products,49.80022506
Warehousing,Distilling rectifying and blending of spirits,0.27708496
Warehousing,Manufacture of wines,29969.95144
Warehousing,Manufacture of beers,9444.573662
Warehousing,Manufacture of soft drinks,29177.64138
Warehousing,Manufacture of tobacco products,0.02748649
Warehousing,Manufacture of textile products,15.41316703
Warehousing,Manufacture of wearing apparel,46.1195627
Warehousing,Manufacture of leather products,0.015717904
Warehousing,Manufacture of footwear,0.2892625
Warehousing,Sawmilling and planing of wood,37.679196
Warehousing,Manufacture of wood products,16.34818944
Warehousing,Manufacture of pulp and paper,29.08691958
Warehousing,Manufacture of containers of paper,4.92056581
Warehousing,Manufacture of other paper products,35.10773864
Warehousing,Printing,20.68568199
Warehousing,Manufacture of refined petroleum products,40.13771139
Warehousing,Manufacture of basic chemicals,34.94321015
Warehousing,Manufacture of paints,10.55454764
Warehousing,Manufacture of pharmaceutical products,65.23919421
Warehousing,Manufacture of soap detergents and toilet products,49.4783291
Warehousing,Manufacture of other chemical products,2.82264296
Warehousing,Manufacture of rubber products,1.526530617
Warehousing,Manufacture of plastic products,4.246925827
Warehousing,Manufacture of glass and glass products,9.540414088
Warehousing,Manufacture of cement,8.745079282
Warehousing,Manufacture of concrete and concrete products,28.30858386
Warehousing,Manufacture of basic iron and steel,1.314214426
Warehousing,Manufacture of other basic metals,8.732481264
Warehousing,Manufacture of fabricated metal products,94.92234918
Warehousing,Manufacture of industrial and domestic machinery and equipment,3.946381571
Warehousing,Manufacture of electric and electronic machinery and equipment,16.50078462
Warehousing,Manufacture of transport equipment,11.29321068
Warehousing,Manufacture of furniture,59.93032299
Warehousing,Repair and installation of machinery and other manufacturing,5.718947839
Warehousing,Electric power generation,25839.01987
Warehousing,Transmission of electric power,1.557329212
Warehousing,Distribution of electric power,29.13439749
Warehousing,Gas and steam manufacture and supply,4.024585929
Warehousing,Water collection treatment and supply,6.506959694
Warehousing,Waste collection and recycling activities,9.769286017
Warehousing,Construction of residential buildings,5.310086997
Warehousing,Construction of non-residential buildings,1.730237263
Warehousing,Civil engineering,20.84605422
Warehousing,Specialized construction activities,3.490159795
Warehousing,Wholesale and retail trade of motor vehicles,6632.950613
Warehousing,Wholesale trade,178788.5936
Warehousing,Retail trade,27547.24609
Warehousing,Accommodation,4.868143269
Warehousing,Food and beverage service activities,22.30203336
Warehousing,Transport via railways,15616.97413
Warehousing,Other passenger land transport,7256.463261
Warehousing,Freight transport by road,37478.4119
Warehousing,Transport via pipeline,0.41188468
Warehousing,Water transport,40324.70645
Warehousing,Air transport,3849.789127
Warehousing,Warehousing,235.2896469
Warehousing,Support activities for land transport,64.21550305
Warehousing,Other support activities for transport,61137.5903
Warehousing,Postal and courier activities,0
Warehousing,Wireless telecommunications,38.30141293
Warehousing,Wired telecommunications,1357.969089
Warehousing,Other telecommunications activities,1071.142397
Warehousing,Information services activities,623.1054206
Warehousing,Publishing activities,18.52700734
Warehousing,Banking monetary intermediation,513.3979429
Warehousing,Insurance activities,110.523746
Warehousing,Auxiliary financial activities,108.869679
Warehousing,Real estate activities,16.45251728
Warehousing,Housing services,0
Warehousing,Legal and accounting services,22.71795791
Warehousing,Architectural and engineering services,110.8097982
Warehousing,Other professional activities,335.4739432
Warehousing,Rental and leasing activities,37.30180782
Warehousing,Business support activities,194.14701
Warehousing,Public administration,8397.680821
Warehousing,Public education,4.594883367
Warehousing,Private education,15.67583866
Warehousing,Public health activities,84.49177163
Warehousing,Private health activities,1030.64749
Warehousing,Activities of organizations,2316.418171
Warehousing,Artistic and entertainment activities,31.35235836
Warehousing,Other personal services,6.171469003
Support activities for land transport,Cultivation of annual crops,3.267506454
Support activities for land transport,Cultivation of vegetables,1.566690123
Support activities for land transport,Cultivation of grapes,2.645734173
Support activities for land transport,Cultivation of other fruits,5.904537874
Support activities for land transport,Cattle bredding,1.452321937
Support activities for land transport,Pigs breeding,0.355691135
Support activities for land transport,Chicken breeding,0.7287039
Support activities for land transport,Breeding of other animals,0.121810173
Support activities for land transport,Farming services,0.627113993
Support activities for land transport,Forestry,272.6746347
Support activities for land transport,Aquaculture,125.7525663
Support activities for land transport,Extractive fishing,0.568641127
Support activities for land transport,Coal mining,0.764066052
Support activities for land transport,Crude oil and gas mining,50.42068234
Support activities for land transport,Copper mining,652.1019044
Support activities for land transport,Iron mining,26.01556126
Support activities for land transport,Mining of other metals,16.82332495
Support activities for land transport,Other mining activities and mining services,26.21773361
Support activities for land transport,Processing and preserving of meat,16.40391672
Support activities for land transport,Processing and preserving of fishmeal and fish oil,0.122470809
Support activities for land transport,Processing and preserving of fish,5.597450799
Support activities for land transport,Processing and preserving of fruits and vegetables,24.52893861
Support activities for land transport,Manufacture of vegetable and animal oils,3.508578907
Support activities for land transport,Manufacture of dairy products,14.15597577
Support activities for land transport,Manufacture of grain mill products,9.874230259
Support activities for land transport,Manufacture of prepared animal feeds,21.2049438
Support activities for land transport,Manufacture of bakery products,15.95420447
Support activities for land transport,Manufacture of macaroni noodles and other related products,1.488352076
Support activities for land transport,Manufacture of other food products,18.21260681
Support activities for land transport,Distilling rectifying and blending of spirits,1.695291287
Support activities for land transport,Manufacture of wines,59.37991879
Support activities for land transport,Manufacture of beers,1643.688558
Support activities for land transport,Manufacture of soft drinks,2187.549891
Support activities for land transport,Manufacture of tobacco products,7.260868614
Support activities for land transport,Manufacture of textile products,7.130676058
Support activities for land transport,Manufacture of wearing apparel,13.31559013
Support activities for land transport,Manufacture of leather products,0.207536196
Support activities for land transport,Manufacture of footwear,1.04319457
Support activities for land transport,Sawmilling and planing of wood,21.96792938
Support activities for land transport,Manufacture of wood products,15.04209937
Support activities for land transport,Manufacture of pulp and paper,7.623831536
Support activities for land transport,Manufacture of containers of paper,5.348588025
Support activities for land transport,Manufacture of other paper products,8.990039389
Support activities for land transport,Printing,12.0997002
Support activities for land transport,Manufacture of refined petroleum products,9.028316799
Support activities for land transport,Manufacture of basic chemicals,44.12975881
Support activities for land transport,Manufacture of paints,2.788991876
Support activities for land transport,Manufacture of pharmaceutical products,16.19740595
Support activities for land transport,Manufacture of soap detergents and toilet products,11.25429412
Support activities for land transport,Manufacture of other chemical products,1.725372711
Support activities for land transport,Manufacture of rubber products,0.5126037
Support activities for land transport,Manufacture of plastic products,60.86386533
Support activities for land transport,Manufacture of glass and glass products,2.451843479
Support activities for land transport,Manufacture of cement,10.74738244
Support activities for land transport,Manufacture of concrete and concrete products,61.1827386
Support activities for land transport,Manufacture of basic iron and steel,10.97326324
Support activities for land transport,Manufacture of other basic metals,2.321281759
Support activities for land transport,Manufacture of fabricated metal products,48.69502975
Support activities for land transport,Manufacture of industrial and domestic machinery and equipment,9.847759142
Support activities for land transport,Manufacture of electric and electronic machinery and equipment,9.087178708
Support activities for land transport,Manufacture of transport equipment,4.729968695
Support activities for land transport,Manufacture of furniture,17.34151608
Support activities for land transport,Repair and installation of machinery and other manufacturing,42.22485679
Support activities for land transport,Electric power generation,18.74432505
Support activities for land transport,Transmission of electric power,2.599878874
Support activities for land transport,Distribution of electric power,45.59794823
Support activities for land transport,Gas and steam manufacture and supply,25.40682242
Support activities for land transport,Water collection treatment and supply,23.11811056
Support activities for land transport,Waste collection and recycling activities,21.14764124
Support activities for land transport,Construction of residential buildings,306.4931174
Support activities for land transport,Construction of non-residential buildings,165.2261582
Support activities for land transport,Civil engineering,997.3050155
Support activities for land transport,Specialized construction activities,55.04658073
Support activities for land transport,Wholesale and retail trade of motor vehicles,101.0791424
Support activities for land transport,Wholesale trade,2248.760716
Support activities for land transport,Retail trade,732.9004236
Support activities for land transport,Accommodation,35.75491161
Support activities for land transport,Food and beverage service activities,134.7586865
Support activities for land transport,Transport via railways,2.319695203
Support activities for land transport,Other passenger land transport,108792.1255
Support activities for land transport,Freight transport by road,200099.355
Support activities for land transport,Transport via pipeline,0.331264043
Support activities for land transport,Water transport,59.9242027
Support activities for land transport,Air transport,141.1445272
Support activities for land transport,Warehousing,93.62922488
Support activities for land transport,Support activities for land transport,13132.91122
Support activities for land transport,Other support activities for transport,190.6410913
Support activities for land transport,Postal and courier activities,238.3767731
Support activities for land transport,Wireless telecommunications,53.37529474
Support activities for land transport,Wired telecommunications,66.66719456
Support activities for land transport,Other telecommunications activities,95.89938221
Support activities for land transport,Information services activities,221.859632
Support activities for land transport,Publishing activities,73.46698619
Support activities for land transport,Banking monetary intermediation,133.05801
Support activities for land transport,Insurance activities,121.4291225
Support activities for land transport,Auxiliary financial activities,54.14270521
Support activities for land transport,Real estate activities,82.52040836
Support activities for land transport,Housing services,0
Support activities for land transport,Legal and accounting services,134.8261871
Support activities for land transport,Architectural and engineering services,228.6295192
Support activities for land transport,Other professional activities,230.4952717
Support activities for land transport,Rental and leasing activities,255.7393909
Support activities for land transport,Business support activities,227.5687691
Support activities for land transport,Public administration,328.037281
Support activities for land transport,Public education,58.46066736
Support activities for land transport,Private education,29.98457052
Support activities for land transport,Public health activities,73.65013675
Support activities for land transport,Private health activities,501.9523953
Support activities for land transport,Activities of organizations,8.985090918
Support activities for land transport,Artistic and entertainment activities,151.7463024
Support activities for land transport,Other personal services,26.71092689
Other support activities for transport,Cultivation of annual crops,41.57913409
Other support activities for transport,Cultivation of vegetables,21.13355758
Other support activities for transport,Cultivation of grapes,30.58415291
Other support activities for transport,Cultivation of other fruits,89.841838
Other support activities for transport,Cattle bredding,34.77700964
Other support activities for transport,Pigs breeding,15.08507249
Other support activities for transport,Chicken breeding,26.32435441
Other support activities for transport,Breeding of other animals,2.876749201
Other support activities for transport,Farming services,2.496661893
Other support activities for transport,Forestry,77.21384015
Other support activities for transport,Aquaculture,7392.305425
Other support activities for transport,Extractive fishing,19008.30451
Other support activities for transport,Coal mining,0.326445604
Other support activities for transport,Crude oil and gas mining,145.4972776
Other support activities for transport,Copper mining,167048.4523
Other support activities for transport,Iron mining,13924.51877
Other support activities for transport,Mining of other metals,5589.506115
Other support activities for transport,Other mining activities and mining services,12250.81551
Other support activities for transport,Processing and preserving of meat,11834.77618
Other support activities for transport,Processing and preserving of fishmeal and fish oil,7201.536386
Other support activities for transport,Processing and preserving of fish,21808.85773
Other support activities for transport,Processing and preserving of fruits and vegetables,15725.02852
Other support activities for transport,Manufacture of vegetable and animal oils,4980.637297
Other support activities for transport,Manufacture of dairy products,5430.618602
Other support activities for transport,Manufacture of grain mill products,2885.311886
Other support activities for transport,Manufacture of prepared animal feeds,1754.20704
Other support activities for transport,Manufacture of bakery products,1011.815281
Other support activities for transport,Manufacture of macaroni noodles and other related products,130.8376368
Other support activities for transport,Manufacture of other food products,7040.522007
Other support activities for transport,Distilling rectifying and blending of spirits,755.9354722
Other support activities for transport,Manufacture of wines,35154.90631
Other support activities for transport,Manufacture of beers,2630.720185
Other support activities for transport,Manufacture of soft drinks,8325.322536
Other support activities for transport,Manufacture of tobacco products,205.7837189
Other support activities for transport,Manufacture of textile products,4619.370565
Other support activities for transport,Manufacture of wearing apparel,5305.767814
Other support activities for transport,Manufacture of leather products,597.988016
Other support activities for transport,Manufacture of footwear,781.8799965
Other support activities for transport,Sawmilling and planing of wood,31455.5693
Other support activities for transport,Manufacture of wood products,17786.57355
Other support activities for transport,Manufacture of pulp and paper,35725.90853
Other support activities for transport,Manufacture of containers of paper,5953.260723
Other support activities for transport,Manufacture of other paper products,3630.01727
Other support activities for transport,Printing,752.8418958
Other support activities for transport,Manufacture of refined petroleum products,6073.377226
Other support activities for transport,Manufacture of basic chemicals,21597.76802
Other support activities for transport,Manufacture of paints,553.2300057
Other support activities for transport,Manufacture of pharmaceutical products,13965.0019
Other support activities for transport,Manufacture of soap detergents and toilet products,6829.235973
Other support activities for transport,Manufacture of other chemical products,5961.030888
Other support activities for transport,Manufacture of rubber products,3294.301501
Other support activities for transport,Manufacture of plastic products,8734.454888
Other support activities for transport,Manufacture of glass and glass products,4081.603065
Other support activities for transport,Manufacture of cement,152.5480235
Other support activities for transport,Manufacture of concrete and concrete products,4804.742349
Other support activities for transport,Manufacture of basic iron and steel,6566.48882
Other support activities for transport,Manufacture of other basic metals,3096.364021
Other support activities for transport,Manufacture of fabricated metal products,8985.998797
Other support activities for transport,Manufacture of industrial and domestic machinery and equipment,4637.324954
Other support activities for transport,Manufacture of electric and electronic machinery and equipment,3137.10908
Other support activities for transport,Manufacture of transport equipment,908.1537404
Other support activities for transport,Manufacture of furniture,3559.159773
Other support activities for transport,Repair and installation of machinery and other manufacturing,1133.058994
Other support activities for transport,Electric power generation,13555.91594
Other support activities for transport,Transmission of electric power,16.7345462
Other support activities for transport,Distribution of electric power,91.21239369
Other support activities for transport,Gas and steam manufacture and supply,3308.482564
Other support activities for transport,Water collection treatment and supply,47.17108384
Other support activities for transport,Waste collection and recycling activities,129.4913257
Other support activities for transport,Construction of residential buildings,367.3945153
Other support activities for transport,Construction of non-residential buildings,166.6440802
Other support activities for transport,Civil engineering,442.74973
Other support activities for transport,Specialized construction activities,52.33204989
Other support activities for transport,Wholesale and retail trade of motor vehicles,60264.68434
Other support activities for transport,Wholesale trade,408277.589
Other support activities for transport,Retail trade,124578.6471
Other support activities for transport,Accommodation,856.7433935
Other support activities for transport,Food and beverage service activities,1330.212874
Other support activities for transport,Transport via railways,22211.77624
Other support activities for transport,Other passenger land transport,94561.98925
Other support activities for transport,Freight transport by road,26274.74361
Other support activities for transport,Transport via pipeline,6.543919095
Other support activities for transport,Water transport,43627.72074
Other support activities for transport,Air transport,123387.9809
Other support activities for transport,Warehousing,3170.668389
Other support activities for transport,Support activities for land transport,1388.779275
Other support activities for transport,Other support activities for transport,127711.4903
Other support activities for transport,Postal and courier activities,1146.786075
Other support activities for transport,Wireless telecommunications,4392.41622
Other support activities for transport,Wired telecommunications,554.5427848
Other support activities for transport,Other telecommunications activities,367.6006329
Other support activities for transport,Information services activities,2271.234657
Other support activities for transport,Publishing activities,591.3725938
Other support activities for transport,Banking monetary intermediation,1768.187517
Other support activities for transport,Insurance activities,350.3571509
Other support activities for transport,Auxiliary financial activities,223.9863993
Other support activities for transport,Real estate activities,1727.815712
Other support activities for transport,Housing services,0
Other support activities for transport,Legal and accounting services,592.0520998
Other support activities for transport,Architectural and engineering services,1394.548145
Other support activities for transport,Other professional activities,1706.286774
Other support activities for transport,Rental and leasing activities,1335.96875
Other support activities for transport,Business support activities,6315.572296
Other support activities for transport,Public administration,3006.000404
Other support activities for transport,Public education,1470.606817
Other support activities for transport,Private education,1018.624674
Other support activities for transport,Public health activities,267.3504075
Other support activities for transport,Private health activities,1115.210256
Other support activities for transport,Activities of organizations,724.0120629
Other support activities for transport,Artistic and entertainment activities,636.8724701
Other support activities for transport,Other personal services,1075.318997
Postal and courier activities,Cultivation of annual crops,633.8066603
Postal and courier activities,Cultivation of vegetables,491.1592061
Postal and courier activities,Cultivation of grapes,833.2462157
Postal and courier activities,Cultivation of other fruits,2137.55768
Postal and courier activities,Cattle bredding,276.7413645
Postal and courier activities,Pigs breeding,186.6808482
Postal and courier activities,Chicken breeding,322.8770794
Postal and courier activities,Breeding of other animals,41.95845676
Postal and courier activities,Farming services,11.63969302
Postal and courier activities,Forestry,6.862982506
Postal and courier activities,Aquaculture,1709.188654
Postal and courier activities,Extractive fishing,0.095462993
Postal and courier activities,Coal mining,1.625491708
Postal and courier activities,Crude oil and gas mining,4.177506073
Postal and courier activities,Copper mining,141.1018439
Postal and courier activities,Iron mining,1.484581282
Postal and courier activities,Mining of other metals,610.9582819
Postal and courier activities,Other mining activities and mining services,103.0808121
Postal and courier activities,Processing and preserving of meat,10.86946897
Postal and courier activities,Processing and preserving of fishmeal and fish oil,130.0009049
Postal and courier activities,Processing and preserving of fish,5.233365833
Postal and courier activities,Processing and preserving of fruits and vegetables,8.060415983
Postal and courier activities,Manufacture of vegetable and animal oils,151.3758792
Postal and courier activities,Manufacture of dairy products,1035.415895
Postal and courier activities,Manufacture of grain mill products,433.5857173
Postal and courier activities,Manufacture of prepared animal feeds,12.59802695
Postal and courier activities,Manufacture of bakery products,2320.436168
Postal and courier activities,Manufacture of macaroni noodles and other related products,56.33939496
Postal and courier activities,Manufacture of other food products,89.94235016
Postal and courier activities,Distilling rectifying and blending of spirits,222.0017142
Postal and courier activities,Manufacture of wines,26.43904392
Postal and courier activities,Manufacture of beers,267.2163328
Postal and courier activities,Manufacture of soft drinks,70.74591061
Postal and courier activities,Manufacture of tobacco products,0.353313447
Postal and courier activities,Manufacture of textile products,787.0940292
Postal and courier activities,Manufacture of wearing apparel,19.95101861
Postal and courier activities,Manufacture of leather products,67.34008078
Postal and courier activities,Manufacture of footwear,0.807652525
Postal and courier activities,Sawmilling and planing of wood,37.23370988
Postal and courier activities,Manufacture of wood products,533.5857013
Postal and courier activities,Manufacture of pulp and paper,784.6832011
Postal and courier activities,Manufacture of containers of paper,7.02937155
Postal and courier activities,Manufacture of other paper products,20.27798293
Postal and courier activities,Printing,10.87656407
Postal and courier activities,Manufacture of refined petroleum products,412.4148345
Postal and courier activities,Manufacture of basic chemicals,23.72407143
Postal and courier activities,Manufacture of paints,6.308549626
Postal and courier activities,Manufacture of pharmaceutical products,1541.237431
Postal and courier activities,Manufacture of soap detergents and toilet products,24.99970664
Postal and courier activities,Manufacture of other chemical products,5.964038515
Postal and courier activities,Manufacture of rubber products,1.55930816
Postal and courier activities,Manufacture of plastic products,12.55592352
Postal and courier activities,Manufacture of glass and glass products,338.6850471
Postal and courier activities,Manufacture of cement,560.7682184
Postal and courier activities,Manufacture of concrete and concrete products,225.0006556
Postal and courier activities,Manufacture of basic iron and steel,3.877873812
Postal and courier activities,Manufacture of other basic metals,134.5670106
Postal and courier activities,Manufacture of fabricated metal products,2433.29597
Postal and courier activities,Manufacture of industrial and domestic machinery and equipment,13.71715104
Postal and courier activities,Manufacture of electric and electronic machinery and equipment,8.779258437
Postal and courier activities,Manufacture of transport equipment,596.6152313
Postal and courier activities,Manufacture of furniture,29.50952187
Postal and courier activities,Repair and installation of machinery and other manufacturing,591.0379996
Postal and courier activities,Electric power generation,3.163747959
Postal and courier activities,Transmission of electric power,0.723043834
Postal and courier activities,Distribution of electric power,11.57066585
Postal and courier activities,Gas and steam manufacture and supply,6144.904588
Postal and courier activities,Water collection treatment and supply,2.78415501
Postal and courier activities,Waste collection and recycling activities,363.2754568
Postal and courier activities,Construction of residential buildings,1877.590814
Postal and courier activities,Construction of non-residential buildings,953.90306
Postal and courier activities,Civil engineering,1307.328377
Postal and courier activities,Specialized construction activities,271.2879148
Postal and courier activities,Wholesale and retail trade of motor vehicles,3457.766414
Postal and courier activities,Wholesale trade,23292.07331
Postal and courier activities,Retail trade,21219.75408
Postal and courier activities,Accommodation,915.9675935
Postal and courier activities,Food and beverage service activities,2800.127909
Postal and courier activities,Transport via railways,287.7137934
Postal and courier activities,Other passenger land transport,321.7292747
Postal and courier activities,Freight transport by road,2392.073142
Postal and courier activities,Transport via pipeline,0.204878039
Postal and courier activities,Water transport,4.031140201
Postal and courier activities,Air transport,105.1143924
Postal and courier activities,Warehousing,10.0067652
Postal and courier activities,Support activities for land transport,1898.03232
Postal and courier activities,Other support activities for transport,2433.880127
Postal and courier activities,Postal and courier activities,8.49550375
Postal and courier activities,Wireless telecommunications,10886.48778
Postal and courier activities,Wired telecommunications,7710.264663
Postal and courier activities,Other telecommunications activities,8.788914046
Postal and courier activities,Information services activities,6762.969723
Postal and courier activities,Publishing activities,3322.411344
Postal and courier activities,Banking monetary intermediation,11323.40584
Postal and courier activities,Insurance activities,13410.37025
Postal and courier activities,Auxiliary financial activities,6987.580899
Postal and courier activities,Real estate activities,2771.757542
Postal and courier activities,Housing services,0
Postal and courier activities,Legal and accounting services,1229.070953
Postal and courier activities,Architectural and engineering services,5700.646798
Postal and courier activities,Other professional activities,6751.892894
Postal and courier activities,Rental and leasing activities,3342.010036
Postal and courier activities,Business support activities,16438.51533
Postal and courier activities,Public administration,28250.32108
Postal and courier activities,Public education,978.416881
Postal and courier activities,Private education,1961.519453
Postal and courier activities,Public health activities,1750.572846
Postal and courier activities,Private health activities,3296.311803
Postal and courier activities,Activities of organizations,1925.035743
Postal and courier activities,Artistic and entertainment activities,382.1436195
Postal and courier activities,Other personal services,757.1326127
Wireless telecommunications,Cultivation of annual crops,583.2270196
Wireless telecommunications,Cultivation of vegetables,321.4020385
Wireless telecommunications,Cultivation of grapes,501.1335786
Wireless telecommunications,Cultivation of other fruits,1521.935093
Wireless telecommunications,Cattle bredding,343.1942128
Wireless telecommunications,Pigs breeding,176.3602611
Wireless telecommunications,Chicken breeding,322.6126009
Wireless telecommunications,Breeding of other animals,41.25738404
Wireless telecommunications,Farming services,26.63903285
Wireless telecommunications,Forestry,1326.59448
Wireless telecommunications,Aquaculture,1406.10839
Wireless telecommunications,Extractive fishing,1347.874114
Wireless telecommunications,Coal mining,38.67041299
Wireless telecommunications,Crude oil and gas mining,348.6092105
Wireless telecommunications,Copper mining,15799.89577
Wireless telecommunications,Iron mining,279.3867549
Wireless telecommunications,Mining of other metals,688.121391
Wireless telecommunications,Other mining activities and mining services,310.5537737
Wireless telecommunications,Processing and preserving of meat,2465.568102
Wireless telecommunications,Processing and preserving of fishmeal and fish oil,110.6705332
Wireless telecommunications,Processing and preserving of fish,1122.484018
Wireless telecommunications,Processing and preserving of fruits and vegetables,889.3536572
Wireless telecommunications,Manufacture of vegetable and animal oils,111.7929726
Wireless telecommunications,Manufacture of dairy products,1521.207927
Wireless telecommunications,Manufacture of grain mill products,428.3358724
Wireless telecommunications,Manufacture of prepared animal feeds,807.2611152
Wireless telecommunications,Manufacture of bakery products,2170.001118
Wireless telecommunications,Manufacture of macaroni noodles and other related products,114.6122499
Wireless telecommunications,Manufacture of other food products,1580.834743
Wireless telecommunications,Distilling rectifying and blending of spirits,170.0376155
Wireless telecommunications,Manufacture of wines,1289.495226
Wireless telecommunications,Manufacture of beers,278.6738769
Wireless telecommunications,Manufacture of soft drinks,842.9787396
Wireless telecommunications,Manufacture of tobacco products,184.6402036
Wireless telecommunications,Manufacture of textile products,765.2423055
Wireless telecommunications,Manufacture of wearing apparel,1709.276364
Wireless telecommunications,Manufacture of leather products,26.2337351
Wireless telecommunications,Manufacture of footwear,245.0178325
Wireless telecommunications,Sawmilling and planing of wood,1308.443837
Wireless telecommunications,Manufacture of wood products,1415.374031
Wireless telecommunications,Manufacture of pulp and paper,1788.223133
Wireless telecommunications,Manufacture of containers of paper,588.1968059
Wireless telecommunications,Manufacture of other paper products,414.533876
Wireless telecommunications,Printing,1131.966644
Wireless telecommunications,Manufacture of refined petroleum products,692.6592462
Wireless telecommunications,Manufacture of basic chemicals,1762.422528
Wireless telecommunications,Manufacture of paints,439.9902737
Wireless telecommunications,Manufacture of pharmaceutical products,1336.344236
Wireless telecommunications,Manufacture of soap detergents and toilet products,1137.811234
Wireless telecommunications,Manufacture of other chemical products,498.0077008
Wireless telecommunications,Manufacture of rubber products,388.6493658
Wireless telecommunications,Manufacture of plastic products,1756.509243
Wireless telecommunications,Manufacture of glass and glass products,280.8094933
Wireless telecommunications,Manufacture of cement,234.885701
Wireless telecommunications,Manufacture of concrete and concrete products,1853.45835
Wireless telecommunications,Manufacture of basic iron and steel,379.4975226
Wireless telecommunications,Manufacture of other basic metals,217.0577538
Wireless telecommunications,Manufacture of fabricated metal products,3280.858833
Wireless telecommunications,Manufacture of industrial and domestic machinery and equipment,2452.330462
Wireless telecommunications,Manufacture of electric and electronic machinery and equipment,1097.076092
Wireless telecommunications,Manufacture of transport equipment,470.1219669
Wireless telecommunications,Manufacture of furniture,1106.546418
Wireless telecommunications,Repair and installation of machinery and other manufacturing,2144.450915
Wireless telecommunications,Electric power generation,3073.855501
Wireless telecommunications,Transmission of electric power,470.6931722
Wireless telecommunications,Distribution of electric power,2289.152211
Wireless telecommunications,Gas and steam manufacture and supply,462.300687
Wireless telecommunications,Water collection treatment and supply,943.4183725
Wireless telecommunications,Waste collection and recycling activities,943.4666081
Wireless telecommunications,Construction of residential buildings,3315.437221
Wireless telecommunications,Construction of non-residential buildings,2019.21761
Wireless telecommunications,Civil engineering,9099.376952
Wireless telecommunications,Specialized construction activities,2193.447857
Wireless telecommunications,Wholesale and retail trade of motor vehicles,7001.974239
Wireless telecommunications,Wholesale trade,37075.03376
Wireless telecommunications,Retail trade,25589.58894
Wireless telecommunications,Accommodation,4431.015436
Wireless telecommunications,Food and beverage service activities,7017.632789
Wireless telecommunications,Transport via railways,207.3500874
Wireless telecommunications,Other passenger land transport,10529.17114
Wireless telecommunications,Freight transport by road,12219.43724
Wireless telecommunications,Transport via pipeline,170.3443829
Wireless telecommunications,Water transport,583.9981488
Wireless telecommunications,Air transport,3738.456841
Wireless telecommunications,Warehousing,910.2401734
Wireless telecommunications,Support activities for land transport,1485.882423
Wireless telecommunications,Other support activities for transport,3749.78789
Wireless telecommunications,Postal and courier activities,2047.460397
Wireless telecommunications,Wireless telecommunications,334590.5961
Wireless telecommunications,Wired telecommunications,207081.9944
Wireless telecommunications,Other telecommunications activities,254882.0858
Wireless telecommunications,Information services activities,34586.72195
Wireless telecommunications,Publishing activities,3631.375066
Wireless telecommunications,Banking monetary intermediation,17721.38777
Wireless telecommunications,Insurance activities,13164.20075
Wireless telecommunications,Auxiliary financial activities,7666.236887
Wireless telecommunications,Real estate activities,6824.028053
Wireless telecommunications,Housing services,0
Wireless telecommunications,Legal and accounting services,8620.50609
Wireless telecommunications,Architectural and engineering services,11923.12711
Wireless telecommunications,Other professional activities,24537.18099
Wireless telecommunications,Rental and leasing activities,8734.361854
Wireless telecommunications,Business support activities,29695.56307
Wireless telecommunications,Public administration,28075.80088
Wireless telecommunications,Public education,2162.69352
Wireless telecommunications,Private education,5630.264856
Wireless telecommunications,Public health activities,7475.431132
Wireless telecommunications,Private health activities,13776.1967
Wireless telecommunications,Activities of organizations,4616.11348
Wireless telecommunications,Artistic and entertainment activities,3302.866455
Wireless telecommunications,Other personal services,1642.763928
Wired telecommunications,Cultivation of annual crops,416.9635668
Wired telecommunications,Cultivation of vegetables,285.6702486
Wired telecommunications,Cultivation of grapes,433.966392
Wired telecommunications,Cultivation of other fruits,1075.216022
Wired telecommunications,Cattle bredding,333.2987499
Wired telecommunications,Pigs breeding,168.0682696
Wired telecommunications,Chicken breeding,329.4157469
Wired telecommunications,Breeding of other animals,37.51830036
Wired telecommunications,Farming services,19.5546689
Wired telecommunications,Forestry,1409.18054
Wired telecommunications,Aquaculture,1570.130337
Wired telecommunications,Extractive fishing,1158.330413
Wired telecommunications,Coal mining,34.3092159
Wired telecommunications,Crude oil and gas mining,335.8907121
Wired telecommunications,Copper mining,17822.46048
Wired telecommunications,Iron mining,416.0504583
Wired telecommunications,Mining of other metals,723.86754
Wired telecommunications,Other mining activities and mining services,367.5914063
Wired telecommunications,Processing and preserving of meat,2059.813015
Wired telecommunications,Processing and preserving of fishmeal and fish oil,109.0810592
Wired telecommunications,Processing and preserving of fish,949.7156164
Wired telecommunications,Processing and preserving of fruits and vegetables,673.645943
Wired telecommunications,Manufacture of vegetable and animal oils,85.44040979
Wired telecommunications,Manufacture of dairy products,1655.705784
Wired telecommunications,Manufacture of grain mill products,495.4378115
Wired telecommunications,Manufacture of prepared animal feeds,708.7875584
Wired telecommunications,Manufacture of bakery products,1783.832973
Wired telecommunications,Manufacture of macaroni noodles and other related products,108.6242183
Wired telecommunications,Manufacture of other food products,1633.010746
Wired telecommunications,Distilling rectifying and blending of spirits,152.9028428
Wired telecommunications,Manufacture of wines,1496.226678
Wired telecommunications,Manufacture of beers,288.1258989
Wired telecommunications,Manufacture of soft drinks,1170.392453
Wired telecommunications,Manufacture of tobacco products,167.5229614
Wired telecommunications,Manufacture of textile products,689.334704
Wired telecommunications,Manufacture of wearing apparel,1524.282337
Wired telecommunications,Manufacture of leather products,21.98669686
Wired telecommunications,Manufacture of footwear,196.1993225
Wired telecommunications,Sawmilling and planing of wood,1129.537882
Wired telecommunications,Manufacture of wood products,1407.289634
Wired telecommunications,Manufacture of pulp and paper,1900.901427
Wired telecommunications,Manufacture of containers of paper,522.9186273
Wired telecommunications,Manufacture of other paper products,608.8277388
Wired telecommunications,Printing,955.0938253
Wired telecommunications,Manufacture of refined petroleum products,823.2623642
Wired telecommunications,Manufacture of basic chemicals,1626.297112
Wired telecommunications,Manufacture of paints,435.373716
Wired telecommunications,Manufacture of pharmaceutical products,1527.142135
Wired telecommunications,Manufacture of soap detergents and toilet products,1248.709235
Wired telecommunications,Manufacture of other chemical products,459.8918091
Wired telecommunications,Manufacture of rubber products,355.6312928
Wired telecommunications,Manufacture of plastic products,1501.07961
Wired telecommunications,Manufacture of glass and glass products,320.5872961
Wired telecommunications,Manufacture of cement,293.1329626
Wired telecommunications,Manufacture of concrete and concrete products,1586.282978
Wired telecommunications,Manufacture of basic iron and steel,477.6600492
Wired telecommunications,Manufacture of other basic metals,313.2803145
Wired telecommunications,Manufacture of fabricated metal products,3214.83264
Wired telecommunications,Manufacture of industrial and domestic machinery and equipment,1989.030201
Wired telecommunications,Manufacture of electric and electronic machinery and equipment,1065.759128
Wired telecommunications,Manufacture of transport equipment,521.5523854
Wired telecommunications,Manufacture of furniture,1213.629262
Wired telecommunications,Repair and installation of machinery and other manufacturing,1941.95331
Wired telecommunications,Electric power generation,2714.588586
Wired telecommunications,Transmission of electric power,463.2251377
Wired telecommunications,Distribution of electric power,2403.778494
Wired telecommunications,Gas and steam manufacture and supply,469.183171
Wired telecommunications,Water collection treatment and supply,942.1915896
Wired telecommunications,Waste collection and recycling activities,1174.266749
Wired telecommunications,Construction of residential buildings,3141.911415
Wired telecommunications,Construction of non-residential buildings,1772.406667
Wired telecommunications,Civil engineering,8714.695854
Wired telecommunications,Specialized construction activities,1380.828374
Wired telecommunications,Wholesale and retail trade of motor vehicles,6279.884958
Wired telecommunications,Wholesale trade,34184.31867
Wired telecommunications,Retail trade,24686.31513
Wired telecommunications,Accommodation,3938.009899
Wired telecommunications,Food and beverage service activities,5209.913662
Wired telecommunications,Transport via railways,263.7315726
Wired telecommunications,Other passenger land transport,8903.642517
Wired telecommunications,Freight transport by road,11408.24989
Wired telecommunications,Transport via pipeline,175.0234819
Wired telecommunications,Water transport,613.0842045
Wired telecommunications,Air transport,3943.256611
Wired telecommunications,Warehousing,721.6877127
Wired telecommunications,Support activities for land transport,1417.400527
Wired telecommunications,Other support activities for transport,4394.676475
Wired telecommunications,Postal and courier activities,1751.883867
Wired telecommunications,Wireless telecommunications,55691.76665
Wired telecommunications,Wired telecommunications,34270.89798
Wired telecommunications,Other telecommunications activities,53762.54222
Wired telecommunications,Information services activities,31749.6834
Wired telecommunications,Publishing activities,3240.322121
Wired telecommunications,Banking monetary intermediation,18421.07122
Wired telecommunications,Insurance activities,12661.16782
Wired telecommunications,Auxiliary financial activities,8497.076899
Wired telecommunications,Real estate activities,6229.375917
Wired telecommunications,Housing services,0
Wired telecommunications,Legal and accounting services,6516.415302
Wired telecommunications,Architectural and engineering services,10714.56028
Wired telecommunications,Other professional activities,22904.09695
Wired telecommunications,Rental and leasing activities,8295.315297
Wired telecommunications,Business support activities,25744.2711
Wired telecommunications,Public administration,30475.24834
Wired telecommunications,Public education,5589.23999
Wired telecommunications,Private education,6168.960764
Wired telecommunications,Public health activities,8527.093372
Wired telecommunications,Private health activities,13601.09988
Wired telecommunications,Activities of organizations,4417.209883
Wired telecommunications,Artistic and entertainment activities,3274.431396
Wired telecommunications,Other personal services,1439.34197
Other telecommunications activities,Cultivation of annual crops,692.4773888
Other telecommunications activities,Cultivation of vegetables,395.8925874
Other telecommunications activities,Cultivation of grapes,621.7074259
Other telecommunications activities,Cultivation of other fruits,1750.385803
Other telecommunications activities,Cattle bredding,433.872333
Other telecommunications activities,Pigs breeding,219.6836173
Other telecommunications activities,Chicken breeding,413.9154442
Other telecommunications activities,Breeding of other animals,50.47016703
Other telecommunications activities,Farming services,25.72586255
Other telecommunications activities,Forestry,3488.171701
Other telecommunications activities,Aquaculture,2653.219214
Other telecommunications activities,Extractive fishing,1654.225571
Other telecommunications activities,Coal mining,52.20711821
Other telecommunications activities,Crude oil and gas mining,590.2288022
Other telecommunications activities,Copper mining,26258.3992
Other telecommunications activities,Iron mining,548.8616895
Other telecommunications activities,Mining of other metals,1010.55019
Other telecommunications activities,Other mining activities and mining services,590.2264675
Other telecommunications activities,Processing and preserving of meat,3016.340835
Other telecommunications activities,Processing and preserving of fishmeal and fish oil,144.9144894
Other telecommunications activities,Processing and preserving of fish,1442.964699
Other telecommunications activities,Processing and preserving of fruits and vegetables,1133.391083
Other telecommunications activities,Manufacture of vegetable and animal oils,145.69767
Other telecommunications activities,Manufacture of dairy products,2132.984185
Other telecommunications activities,Manufacture of grain mill products,646.98161
Other telecommunications activities,Manufacture of prepared animal feeds,1106.75452
Other telecommunications activities,Manufacture of bakery products,2217.909254
Other telecommunications activities,Manufacture of macaroni noodles and other related products,146.1100484
Other telecommunications activities,Manufacture of other food products,2333.712129
Other telecommunications activities,Distilling rectifying and blending of spirits,243.1644609
Other telecommunications activities,Manufacture of wines,2121.656384
Other telecommunications activities,Manufacture of beers,439.8948187
Other telecommunications activities,Manufacture of soft drinks,1460.39423
Other telecommunications activities,Manufacture of tobacco products,298.8494552
Other telecommunications activities,Manufacture of textile products,1009.921631
Other telecommunications activities,Manufacture of wearing apparel,2069.269069
Other telecommunications activities,Manufacture of leather products,31.41435253
Other telecommunications activities,Manufacture of footwear,275.064235
Other telecommunications activities,Sawmilling and planing of wood,1806.604121
Other telecommunications activities,Manufacture of wood products,1963.822243
Other telecommunications activities,Manufacture of pulp and paper,2510.342858
Other telecommunications activities,Manufacture of containers of paper,743.0257274
Other telecommunications activities,Manufacture of other paper products,702.3427413
Other telecommunications activities,Printing,1453.659286
Other telecommunications activities,Manufacture of refined petroleum products,1038.963086
Other telecommunications activities,Manufacture of basic chemicals,2521.20115
Other telecommunications activities,Manufacture of paints,603.3818047
Other telecommunications activities,Manufacture of pharmaceutical products,2121.012183
Other telecommunications activities,Manufacture of soap detergents and toilet products,1643.927287
Other telecommunications activities,Manufacture of other chemical products,677.6865779
Other telecommunications activities,Manufacture of rubber products,487.9500833
Other telecommunications activities,Manufacture of plastic products,2269.6309
Other telecommunications activities,Manufacture of glass and glass products,397.3508983
Other telecommunications activities,Manufacture of cement,433.1875382
Other telecommunications activities,Manufacture of concrete and concrete products,2734.698068
Other telecommunications activities,Manufacture of basic iron and steel,637.6836006
Other telecommunications activities,Manufacture of other basic metals,351.9290406
Other telecommunications activities,Manufacture of fabricated metal products,4725.943051
Other telecommunications activities,Manufacture of industrial and domestic machinery and equipment,2984.408017
Other telecommunications activities,Manufacture of electric and electronic machinery and equipment,1458.397489
Other telecommunications activities,Manufacture of transport equipment,672.9732489
Other telecommunications activities,Manufacture of furniture,1520.075851
Other telecommunications activities,Repair and installation of machinery and other manufacturing,2925.111604
Other telecommunications activities,Electric power generation,4016.940599
Other telecommunications activities,Transmission of electric power,623.6894743
Other telecommunications activities,Distribution of electric power,3293.397832
Other telecommunications activities,Gas and steam manufacture and supply,941.1039309
Other telecommunications activities,Water collection treatment and supply,1361.294055
Other telecommunications activities,Waste collection and recycling activities,1469.301329
Other telecommunications activities,Construction of residential buildings,6312.748697
Other telecommunications activities,Construction of non-residential buildings,3466.324124
Other telecommunications activities,Civil engineering,17803.43702
Other telecommunications activities,Specialized construction activities,2344.609229
Other telecommunications activities,Wholesale and retail trade of motor vehicles,8905.524202
Other telecommunications activities,Wholesale trade,48388.19272
Other telecommunications activities,Retail trade,33110.47493
Other telecommunications activities,Accommodation,5670.756415
Other telecommunications activities,Food and beverage service activities,7824.810025
Other telecommunications activities,Transport via railways,313.270377
Other telecommunications activities,Other passenger land transport,12354.82102
Other telecommunications activities,Freight transport by road,19521.31755
Other telecommunications activities,Transport via pipeline,225.1438648
Other telecommunications activities,Water transport,1148.080562
Other telecommunications activities,Air transport,5674.431261
Other telecommunications activities,Warehousing,1149.923724
Other telecommunications activities,Support activities for land transport,2049.584472
Other telecommunications activities,Other support activities for transport,6794.741688
Other telecommunications activities,Postal and courier activities,2858.76098
Other telecommunications activities,Wireless telecommunications,44495.05437
Other telecommunications activities,Wired telecommunications,26435.29112
Other telecommunications activities,Other telecommunications activities,53119.23103
Other telecommunications activities,Information services activities,44031.6026
Other telecommunications activities,Publishing activities,5207.238382
Other telecommunications activities,Banking monetary intermediation,27601.71459
Other telecommunications activities,Insurance activities,18343.36344
Other telecommunications activities,Auxiliary financial activities,11918.73742
Other telecommunications activities,Real estate activities,8774.496259
Other telecommunications activities,Housing services,0
Other telecommunications activities,Legal and accounting services,9881.206476
Other telecommunications activities,Architectural and engineering services,15835.82305
Other telecommunications activities,Other professional activities,33690.40179
Other telecommunications activities,Rental and leasing activities,12834.02919
Other telecommunications activities,Business support activities,37400.80544
Other telecommunications activities,Public administration,43093.14662
Other telecommunications activities,Public education,6361.095459
Other telecommunications activities,Private education,8649.877509
Other telecommunications activities,Public health activities,10654.96409
Other telecommunications activities,Private health activities,17749.43597
Other telecommunications activities,Activities of organizations,6442.850237
Other telecommunications activities,Artistic and entertainment activities,5354.705503
Other telecommunications activities,Other personal services,2129.373262
Information services activities,Cultivation of annual crops,388.4488981
Information services activities,Cultivation of vegetables,245.0730767
Information services activities,Cultivation of grapes,464.9636559
Information services activities,Cultivation of other fruits,912.8522698
Information services activities,Cattle bredding,344.8242777
Information services activities,Pigs breeding,53.56061393
Information services activities,Chicken breeding,66.59750516
Information services activities,Breeding of other animals,26.12062426
Information services activities,Farming services,184.9343045
Information services activities,Forestry,404.8331695
Information services activities,Aquaculture,47.40412095
Information services activities,Extractive fishing,91.84690862
Information services activities,Coal mining,2.962420159
Information services activities,Crude oil and gas mining,298.1574942
Information services activities,Copper mining,76694.46205
Information services activities,Iron mining,38.56918465
Information services activities,Mining of other metals,183.201265
Information services activities,Other mining activities and mining services,123.7331887
Information services activities,Processing and preserving of meat,3354.586251
Information services activities,Processing and preserving of fishmeal and fish oil,181.849276
Information services activities,Processing and preserving of fish,826.2791043
Information services activities,Processing and preserving of fruits and vegetables,218.7946336
Information services activities,Manufacture of vegetable and animal oils,122.5829064
Information services activities,Manufacture of dairy products,18645.14471
Information services activities,Manufacture of grain mill products,6974.814622
Information services activities,Manufacture of prepared animal feeds,4025.041412
Information services activities,Manufacture of bakery products,12325.22603
Information services activities,Manufacture of macaroni noodles and other related products,185.0101786
Information services activities,Manufacture of other food products,15489.75189
Information services activities,Distilling rectifying and blending of spirits,104.4481251
Information services activities,Manufacture of wines,16518.82935
Information services activities,Manufacture of beers,3255.694394
Information services activities,Manufacture of soft drinks,22941.22585
Information services activities,Manufacture of tobacco products,23.5438827
Information services activities,Manufacture of textile products,4801.957073
Information services activities,Manufacture of wearing apparel,14358.25509
Information services activities,Manufacture of leather products,7.65459066
Information services activities,Manufacture of footwear,98.76759206
Information services activities,Sawmilling and planing of wood,11723.14723
Information services activities,Manufacture of wood products,5132.682007
Information services activities,Manufacture of pulp and paper,9094.99288
Information services activities,Manufacture of containers of paper,1558.54492
Information services activities,Manufacture of other paper products,10934.31929
Information services activities,Printing,6441.534003
Information services activities,Manufacture of refined petroleum products,12463.7953
Information services activities,Manufacture of basic chemicals,10879.1821
Information services activities,Manufacture of paints,3293.578339
Information services activities,Manufacture of pharmaceutical products,20338.25465
Information services activities,Manufacture of soap detergents and toilet products,15382.10175
Information services activities,Manufacture of other chemical products,924.3269456
Information services activities,Manufacture of rubber products,481.772062
Information services activities,Manufacture of plastic products,1433.147892
Information services activities,Manufacture of glass and glass products,2977.231268
Information services activities,Manufacture of cement,2749.559134
Information services activities,Manufacture of concrete and concrete products,8887.052092
Information services activities,Manufacture of basic iron and steel,500.3685627
Information services activities,Manufacture of other basic metals,2743.558622
Information services activities,Manufacture of fabricated metal products,29535.09102
Information services activities,Manufacture of industrial and domestic machinery and equipment,1349.182655
Information services activities,Manufacture of electric and electronic machinery and equipment,5148.318763
Information services activities,Manufacture of transport equipment,3540.370607
Information services activities,Manufacture of furniture,18614.33354
Information services activities,Repair and installation of machinery and other manufacturing,1818.446854
Information services activities,Electric power generation,1306.470799
Information services activities,Transmission of electric power,482.0264771
Information services activities,Distribution of electric power,9012.373528
Information services activities,Gas and steam manufacture and supply,1358.577891
Information services activities,Water collection treatment and supply,2019.93198
Information services activities,Waste collection and recycling activities,3028.633726
Information services activities,Construction of residential buildings,1919.34949
Information services activities,Construction of non-residential buildings,692.2716833
Information services activities,Civil engineering,6751.861368
Information services activities,Specialized construction activities,1407.488483
Information services activities,Wholesale and retail trade of motor vehicles,10161.84903
Information services activities,Wholesale trade,86480.16133
Information services activities,Retail trade,75939.58102
Information services activities,Accommodation,1542.371827
Information services activities,Food and beverage service activities,7085.286777
Information services activities,Transport via railways,978.5274727
Information services activities,Other passenger land transport,1604.163294
Information services activities,Freight transport by road,3573.520084
Information services activities,Transport via pipeline,127.3951349
Information services activities,Water transport,1275.867755
Information services activities,Air transport,24708.20062
Information services activities,Warehousing,262.9583174
Information services activities,Support activities for land transport,3578.788852
Information services activities,Other support activities for transport,17561.48483
Information services activities,Postal and courier activities,2.830411713
Information services activities,Wireless telecommunications,12141.91344
Information services activities,Wired telecommunications,38312.16844
Information services activities,Other telecommunications activities,6723.100839
Information services activities,Information services activities,192838.3737
Information services activities,Publishing activities,5823.740632
Information services activities,Banking monetary intermediation,159032.4038
Information services activities,Insurance activities,34341.1901
Information services activities,Auxiliary financial activities,31670.36789
Information services activities,Real estate activities,5249.32814
Information services activities,Housing services,0
Information services activities,Legal and accounting services,7129.237257
Information services activities,Architectural and engineering services,34488.4608
Information services activities,Other professional activities,104205.2919
Information services activities,Rental and leasing activities,11633.72721
Information services activities,Business support activities,60482.99815
Information services activities,Public administration,84097.07277
Information services activities,Public education,1502.975624
Information services activities,Private education,5007.843439
Information services activities,Public health activities,26402.07438
Information services activities,Private health activities,3363.678143
Information services activities,Activities of organizations,1038.145624
Information services activities,Artistic and entertainment activities,9758.123822
Information services activities,Other personal services,1942.915406
Publishing activities,Cultivation of annual crops,563.3886736
Publishing activities,Cultivation of vegetables,197.4416265
Publishing activities,Cultivation of grapes,297.8461941
Publishing activities,Cultivation of other fruits,791.1463709
Publishing activities,Cattle bredding,646.5226373
Publishing activities,Pigs breeding,255.8366456
Publishing activities,Chicken breeding,468.3598998
Publishing activities,Breeding of other animals,63.65650066
Publishing activities,Farming services,47.69622884
Publishing activities,Forestry,111.5367778
Publishing activities,Aquaculture,309.917094
Publishing activities,Extractive fishing,14.14648205
Publishing activities,Coal mining,10.52774455
Publishing activities,Crude oil and gas mining,20.31513342
Publishing activities,Copper mining,15813.22122
Publishing activities,Iron mining,16.79834406
Publishing activities,Mining of other metals,52.59181028
Publishing activities,Other mining activities and mining services,160.2904564
Publishing activities,Processing and preserving of meat,10237.88655
Publishing activities,Processing and preserving of fishmeal and fish oil,463.9912714
Publishing activities,Processing and preserving of fish,7007.186155
Publishing activities,Processing and preserving of fruits and vegetables,3605.789423
Publishing activities,Manufacture of vegetable and animal oils,867.8005607
Publishing activities,Manufacture of dairy products,5044.594338
Publishing activities,Manufacture of grain mill products,3058.146165
Publishing activities,Manufacture of prepared animal feeds,4059.917581
Publishing activities,Manufacture of bakery products,3828.650849
Publishing activities,Manufacture of macaroni noodles and other related products,76.89096494
Publishing activities,Manufacture of other food products,4841.594676
Publishing activities,Distilling rectifying and blending of spirits,1454.687347
Publishing activities,Manufacture of wines,9736.68227
Publishing activities,Manufacture of beers,4642.286265
Publishing activities,Manufacture of soft drinks,8666.456312
Publishing activities,Manufacture of tobacco products,1252.292067
Publishing activities,Manufacture of textile products,1515.739345
Publishing activities,Manufacture of wearing apparel,4217.045753
Publishing activities,Manufacture of leather products,116.0708422
Publishing activities,Manufacture of footwear,372.6307949
Publishing activities,Sawmilling and planing of wood,6131.393332
Publishing activities,Manufacture of wood products,5705.797648
Publishing activities,Manufacture of pulp and paper,5492.706204
Publishing activities,Manufacture of containers of paper,2098.715587
Publishing activities,Manufacture of other paper products,4068.495039
Publishing activities,Printing,2988.345587
Publishing activities,Manufacture of refined petroleum products,3986.500905
Publishing activities,Manufacture of basic chemicals,4367.134333
Publishing activities,Manufacture of paints,1550.809106
Publishing activities,Manufacture of pharmaceutical products,12934.95898
Publishing activities,Manufacture of soap detergents and toilet products,5068.342646
Publishing activities,Manufacture of other chemical products,2933.026357
Publishing activities,Manufacture of rubber products,62.87510165
Publishing activities,Manufacture of plastic products,2944.184151
Publishing activities,Manufacture of glass and glass products,678.3379453
Publishing activities,Manufacture of cement,2599.043811
Publishing activities,Manufacture of concrete and concrete products,7227.920084
Publishing activities,Manufacture of basic iron and steel,1762.979934
Publishing activities,Manufacture of other basic metals,771.725063
Publishing activities,Manufacture of fabricated metal products,8769.555619
Publishing activities,Manufacture of industrial and domestic machinery and equipment,4245.161122
Publishing activities,Manufacture of electric and electronic machinery and equipment,1379.839914
Publishing activities,Manufacture of transport equipment,2598.490988
Publishing activities,Manufacture of furniture,2301.074002
Publishing activities,Repair and installation of machinery and other manufacturing,1840.391608
Publishing activities,Electric power generation,3435.373879
Publishing activities,Transmission of electric power,35.21774039
Publishing activities,Distribution of electric power,281.9634759
Publishing activities,Gas and steam manufacture and supply,10195.36901
Publishing activities,Water collection treatment and supply,547.1900786
Publishing activities,Waste collection and recycling activities,332.6193471
Publishing activities,Construction of residential buildings,10117.77273
Publishing activities,Construction of non-residential buildings,3572.735723
Publishing activities,Civil engineering,453.4061627
Publishing activities,Specialized construction activities,1187.10045
Publishing activities,Wholesale and retail trade of motor vehicles,8248.967472
Publishing activities,Wholesale trade,52817.18477
Publishing activities,Retail trade,35006.38091
Publishing activities,Accommodation,775.6997625
Publishing activities,Food and beverage service activities,3690.010723
Publishing activities,Transport via railways,571.3950542
Publishing activities,Other passenger land transport,1289.472315
Publishing activities,Freight transport by road,1675.40214
Publishing activities,Transport via pipeline,1.447542211
Publishing activities,Water transport,310.057584
Publishing activities,Air transport,3268.273243
Publishing activities,Warehousing,1471.517305
Publishing activities,Support activities for land transport,2451.710325
Publishing activities,Other support activities for transport,717.2869339
Publishing activities,Postal and courier activities,190.3052931
Publishing activities,Wireless telecommunications,6909.960146
Publishing activities,Wired telecommunications,4760.319679
Publishing activities,Other telecommunications activities,8773.316807
Publishing activities,Information services activities,14013.90424
Publishing activities,Publishing activities,62717.14422
Publishing activities,Banking monetary intermediation,31433.37695
Publishing activities,Insurance activities,21464.73552
Publishing activities,Auxiliary financial activities,8610.348236
Publishing activities,Real estate activities,12820.82843
Publishing activities,Housing services,0
Publishing activities,Legal and accounting services,9300.01075
Publishing activities,Architectural and engineering services,18115.69921
Publishing activities,Other professional activities,70839.20401
Publishing activities,Rental and leasing activities,5951.432559
Publishing activities,Business support activities,28803.44989
Publishing activities,Public administration,31341.64442
Publishing activities,Public education,12492.03027
Publishing activities,Private education,21424.59727
Publishing activities,Public health activities,4886.87817
Publishing activities,Private health activities,8214.844744
Publishing activities,Activities of organizations,327.3165109
Publishing activities,Artistic and entertainment activities,3375.979438
Publishing activities,Other personal services,260.2741766
Banking monetary intermediation,Cultivation of annual crops,57863.057
Banking monetary intermediation,Cultivation of vegetables,15510.61731
Banking monetary intermediation,Cultivation of grapes,42556.56086
Banking monetary intermediation,Cultivation of other fruits,63184.39402
Banking monetary intermediation,Cattle bredding,27201.54218
Banking monetary intermediation,Pigs breeding,6150.965319
Banking monetary intermediation,Chicken breeding,10254.88227
Banking monetary intermediation,Breeding of other animals,3248.860929
Banking monetary intermediation,Farming services,26348.74935
Banking monetary intermediation,Forestry,10853.24798
Banking monetary intermediation,Aquaculture,61106.30637
Banking monetary intermediation,Extractive fishing,1979.279824
Banking monetary intermediation,Coal mining,336.6206887
Banking monetary intermediation,Crude oil and gas mining,18562.63292
Banking monetary intermediation,Copper mining,99979.34235
Banking monetary intermediation,Iron mining,13375.62569
Banking monetary intermediation,Mining of other metals,2365.022304
Banking monetary intermediation,Other mining activities and mining services,13652.61661
Banking monetary intermediation,Processing and preserving of meat,21988.32512
Banking monetary intermediation,Processing and preserving of fishmeal and fish oil,8098.529741
Banking monetary intermediation,Processing and preserving of fish,35632.59108
Banking monetary intermediation,Processing and preserving of fruits and vegetables,24769.6026
Banking monetary intermediation,Manufacture of vegetable and animal oils,6342.685349
Banking monetary intermediation,Manufacture of dairy products,15927.59858
Banking monetary intermediation,Manufacture of grain mill products,11585.36309
Banking monetary intermediation,Manufacture of prepared animal feeds,4873.223543
Banking monetary intermediation,Manufacture of bakery products,15975.39188
Banking monetary intermediation,Manufacture of macaroni noodles and other related products,2028.853427
Banking monetary intermediation,Manufacture of other food products,13773.80739
Banking monetary intermediation,Distilling rectifying and blending of spirits,3928.846186
Banking monetary intermediation,Manufacture of wines,40773.23241
Banking monetary intermediation,Manufacture of beers,2244.463303
Banking monetary intermediation,Manufacture of soft drinks,6046.838085
Banking monetary intermediation,Manufacture of tobacco products,13144.16526
Banking monetary intermediation,Manufacture of textile products,23045.65737
Banking monetary intermediation,Manufacture of wearing apparel,11340.61242
Banking monetary intermediation,Manufacture of leather products,994.6800323
Banking monetary intermediation,Manufacture of footwear,4012.792381
Banking monetary intermediation,Sawmilling and planing of wood,17319.42626
Banking monetary intermediation,Manufacture of wood products,16864.15909
Banking monetary intermediation,Manufacture of pulp and paper,43659.98826
Banking monetary intermediation,Manufacture of containers of paper,6743.470455
Banking monetary intermediation,Manufacture of other paper products,4696.36124
Banking monetary intermediation,Printing,15190.0698
Banking monetary intermediation,Manufacture of refined petroleum products,49581.16843
Banking monetary intermediation,Manufacture of basic chemicals,40199.66631
Banking monetary intermediation,Manufacture of paints,2350.035849
Banking monetary intermediation,Manufacture of pharmaceutical products,10983.80833
Banking monetary intermediation,Manufacture of soap detergents and toilet products,4325.047755
Banking monetary intermediation,Manufacture of other chemical products,3458.783274
Banking monetary intermediation,Manufacture of rubber products,16395.64306
Banking monetary intermediation,Manufacture of plastic products,37679.44607
Banking monetary intermediation,Manufacture of glass and glass products,3405.989726
Banking monetary intermediation,Manufacture of cement,20241.53915
Banking monetary intermediation,Manufacture of concrete and concrete products,8433.244906
Banking monetary intermediation,Manufacture of basic iron and steel,15365.03335
Banking monetary intermediation,Manufacture of other basic metals,8615.813091
Banking monetary intermediation,Manufacture of fabricated metal products,34243.10885
Banking monetary intermediation,Manufacture of industrial and domestic machinery and equipment,27994.80565
Banking monetary intermediation,Manufacture of electric and electronic machinery and equipment,9554.756549
Banking monetary intermediation,Manufacture of transport equipment,10168.06415
Banking monetary intermediation,Manufacture of furniture,12674.73942
Banking monetary intermediation,Repair and installation of machinery and other manufacturing,17170.32662
Banking monetary intermediation,Electric power generation,107464.3704
Banking monetary intermediation,Transmission of electric power,4776.398782
Banking monetary intermediation,Distribution of electric power,28021.71689
Banking monetary intermediation,Gas and steam manufacture and supply,10913.50231
Banking monetary intermediation,Water collection treatment and supply,18395.08187
Banking monetary intermediation,Waste collection and recycling activities,13669.66186
Banking monetary intermediation,Construction of residential buildings,247110.8507
Banking monetary intermediation,Construction of non-residential buildings,143834.8286
Banking monetary intermediation,Civil engineering,88938.88505
Banking monetary intermediation,Specialized construction activities,46675.76423
Banking monetary intermediation,Wholesale and retail trade of motor vehicles,118142.1935
Banking monetary intermediation,Wholesale trade,477699.3355
Banking monetary intermediation,Retail trade,234719.8273
Banking monetary intermediation,Accommodation,40296.43289
Banking monetary intermediation,Food and beverage service activities,46249.21469
Banking monetary intermediation,Transport via railways,3134.168853
Banking monetary intermediation,Other passenger land transport,31350.41144
Banking monetary intermediation,Freight transport by road,93715.29144
Banking monetary intermediation,Transport via pipeline,1877.858016
Banking monetary intermediation,Water transport,21684.54044
Banking monetary intermediation,Air transport,29423.88832
Banking monetary intermediation,Warehousing,10433.53194
Banking monetary intermediation,Support activities for land transport,114154.936
Banking monetary intermediation,Other support activities for transport,44528.74146
Banking monetary intermediation,Postal and courier activities,2451.131957
Banking monetary intermediation,Wireless telecommunications,19016.66052
Banking monetary intermediation,Wired telecommunications,8248.764609
Banking monetary intermediation,Other telecommunications activities,9569.771875
Banking monetary intermediation,Information services activities,17931.37495
Banking monetary intermediation,Publishing activities,15729.13209
Banking monetary intermediation,Banking monetary intermediation,186200.3069
Banking monetary intermediation,Insurance activities,245968.6192
Banking monetary intermediation,Auxiliary financial activities,181600.0418
Banking monetary intermediation,Real estate activities,475714.4568
Banking monetary intermediation,Housing services,240963.5795
Banking monetary intermediation,Legal and accounting services,17849.22512
Banking monetary intermediation,Architectural and engineering services,46634.47593
Banking monetary intermediation,Other professional activities,94307.96135
Banking monetary intermediation,Rental and leasing activities,64492.55878
Banking monetary intermediation,Business support activities,87302.34524
Banking monetary intermediation,Public administration,9345.948248
Banking monetary intermediation,Public education,3470.883821
Banking monetary intermediation,Private education,54695.81753
Banking monetary intermediation,Public health activities,4972.044031
Banking monetary intermediation,Private health activities,62633.2204
Banking monetary intermediation,Activities of organizations,11378.40757
Banking monetary intermediation,Artistic and entertainment activities,18551.47338
Banking monetary intermediation,Other personal services,14768.43131
Insurance activities,Cultivation of annual crops,2642.756763
Insurance activities,Cultivation of vegetables,2069.018826
Insurance activities,Cultivation of grapes,1040.610449
Insurance activities,Cultivation of other fruits,4562.10198
Insurance activities,Cattle bredding,572.1391263
Insurance activities,Pigs breeding,246.7173911
Insurance activities,Chicken breeding,429.701723
Insurance activities,Breeding of other animals,51.30412108
Insurance activities,Farming services,804.0445646
Insurance activities,Forestry,6.881038411
Insurance activities,Aquaculture,10521.17651
Insurance activities,Extractive fishing,220.3198185
Insurance activities,Coal mining,10.45923726
Insurance activities,Crude oil and gas mining,640.867018
Insurance activities,Copper mining,44901.83925
Insurance activities,Iron mining,4368.292296
Insurance activities,Mining of other metals,1728.674135
Insurance activities,Other mining activities and mining services,565.5887333
Insurance activities,Processing and preserving of meat,3290.126746
Insurance activities,Processing and preserving of fishmeal and fish oil,333.8079692
Insurance activities,Processing and preserving of fish,2113.889266
Insurance activities,Processing and preserving of fruits and vegetables,2809.457278
Insurance activities,Manufacture of vegetable and animal oils,390.3844889
Insurance activities,Manufacture of dairy products,960.749921
Insurance activities,Manufacture of grain mill products,1768.498384
Insurance activities,Manufacture of prepared animal feeds,2501.816508
Insurance activities,Manufacture of bakery products,1413.569981
Insurance activities,Manufacture of macaroni noodles and other related products,613.9375746
Insurance activities,Manufacture of other food products,1185.264675
Insurance activities,Distilling rectifying and blending of spirits,416.5590439
Insurance activities,Manufacture of wines,3720.024994
Insurance activities,Manufacture of beers,679.3354111
Insurance activities,Manufacture of soft drinks,2018.756916
Insurance activities,Manufacture of tobacco products,144.2803663
Insurance activities,Manufacture of textile products,1164.928876
Insurance activities,Manufacture of wearing apparel,2064.668473
Insurance activities,Manufacture of leather products,116.1770772
Insurance activities,Manufacture of footwear,508.0196845
Insurance activities,Sawmilling and planing of wood,2991.62632
Insurance activities,Manufacture of wood products,3696.715107
Insurance activities,Manufacture of pulp and paper,6301.880584
Insurance activities,Manufacture of containers of paper,1378.294722
Insurance activities,Manufacture of other paper products,907.1160743
Insurance activities,Printing,1641.367866
Insurance activities,Manufacture of refined petroleum products,68.855383
Insurance activities,Manufacture of basic chemicals,5884.986061
Insurance activities,Manufacture of paints,818.1510131
Insurance activities,Manufacture of pharmaceutical products,2355.73971
Insurance activities,Manufacture of soap detergents and toilet products,1148.498521
Insurance activities,Manufacture of other chemical products,2484.639844
Insurance activities,Manufacture of rubber products,463.5340579
Insurance activities,Manufacture of plastic products,4350.646002
Insurance activities,Manufacture of glass and glass products,1193.713602
Insurance activities,Manufacture of cement,1778.490549
Insurance activities,Manufacture of concrete and concrete products,2417.367167
Insurance activities,Manufacture of basic iron and steel,2041.201564
Insurance activities,Manufacture of other basic metals,437.4912497
Insurance activities,Manufacture of fabricated metal products,3876.670526
Insurance activities,Manufacture of industrial and domestic machinery and equipment,2283.84316
Insurance activities,Manufacture of electric and electronic machinery and equipment,1501.066605
Insurance activities,Manufacture of transport equipment,1049.441437
Insurance activities,Manufacture of furniture,1827.698487
Insurance activities,Repair and installation of machinery and other manufacturing,1315.733244
Insurance activities,Electric power generation,20841.62368
Insurance activities,Transmission of electric power,1021.446795
Insurance activities,Distribution of electric power,1544.528661
Insurance activities,Gas and steam manufacture and supply,671.7266563
Insurance activities,Water collection treatment and supply,2999.167671
Insurance activities,Waste collection and recycling activities,2173.938589
Insurance activities,Construction of residential buildings,9729.606085
Insurance activities,Construction of non-residential buildings,7089.205545
Insurance activities,Civil engineering,6796.014939
Insurance activities,Specialized construction activities,1498.850513
Insurance activities,Wholesale and retail trade of motor vehicles,9759.232306
Insurance activities,Wholesale trade,60338.70528
Insurance activities,Retail trade,46273.2163
Insurance activities,Accommodation,4047.22853
Insurance activities,Food and beverage service activities,8445.597478
Insurance activities,Transport via railways,1846.312483
Insurance activities,Other passenger land transport,9517.760163
Insurance activities,Freight transport by road,36638.77926
Insurance activities,Transport via pipeline,659.2554935
Insurance activities,Water transport,3243.231082
Insurance activities,Air transport,11515.63002
Insurance activities,Warehousing,718.8989529
Insurance activities,Support activities for land transport,10281.38582
Insurance activities,Other support activities for transport,12783.05588
Insurance activities,Postal and courier activities,56.08600109
Insurance activities,Wireless telecommunications,4924.437038
Insurance activities,Wired telecommunications,738.3252255
Insurance activities,Other telecommunications activities,168.3376483
Insurance activities,Information services activities,4308.94064
Insurance activities,Publishing activities,2999.666307
Insurance activities,Banking monetary intermediation,20000.57401
Insurance activities,Insurance activities,49868.26662
Insurance activities,Auxiliary financial activities,6062.841315
Insurance activities,Real estate activities,15208.31089
Insurance activities,Housing services,4043.177589
Insurance activities,Legal and accounting services,3082.731606
Insurance activities,Architectural and engineering services,15169.51026
Insurance activities,Other professional activities,7073.042989
Insurance activities,Rental and leasing activities,22777.48032
Insurance activities,Business support activities,16450.4939
Insurance activities,Public administration,8637.220994
Insurance activities,Public education,8507.021192
Insurance activities,Private education,10281.56658
Insurance activities,Public health activities,3711.898246
Insurance activities,Private health activities,8891.851749
Insurance activities,Activities of organizations,2235.194722
Insurance activities,Artistic and entertainment activities,4697.109293
Insurance activities,Other personal services,1274.160348
Auxiliary financial activities,Cultivation of annual crops,7690.564427
Auxiliary financial activities,Cultivation of vegetables,3927.750531
Auxiliary financial activities,Cultivation of grapes,5027.162725
Auxiliary financial activities,Cultivation of other fruits,9920.832384
Auxiliary financial activities,Cattle bredding,2465.597357
Auxiliary financial activities,Pigs breeding,1873.643283
Auxiliary financial activities,Chicken breeding,3471.554088
Auxiliary financial activities,Breeding of other animals,680.63129
Auxiliary financial activities,Farming services,6083.049536
Auxiliary financial activities,Forestry,2948.708634
Auxiliary financial activities,Aquaculture,4964.690864
Auxiliary financial activities,Extractive fishing,885.3722471
Auxiliary financial activities,Coal mining,204.0821877
Auxiliary financial activities,Crude oil and gas mining,1366.529127
Auxiliary financial activities,Copper mining,584.832726
Auxiliary financial activities,Iron mining,37.32641942
Auxiliary financial activities,Mining of other metals,503.3162399
Auxiliary financial activities,Other mining activities and mining services,2.690167622
Auxiliary financial activities,Processing and preserving of meat,1563.285379
Auxiliary financial activities,Processing and preserving of fishmeal and fish oil,69.76654021
Auxiliary financial activities,Processing and preserving of fish,2558.09086
Auxiliary financial activities,Processing and preserving of fruits and vegetables,2136.441607
Auxiliary financial activities,Manufacture of vegetable and animal oils,315.8670469
Auxiliary financial activities,Manufacture of dairy products,1590.163425
Auxiliary financial activities,Manufacture of grain mill products,563.9188881
Auxiliary financial activities,Manufacture of prepared animal feeds,2138.610283
Auxiliary financial activities,Manufacture of bakery products,1490.960253
Auxiliary financial activities,Manufacture of macaroni noodles and other related products,100.5891722
Auxiliary financial activities,Manufacture of other food products,991.1367276
Auxiliary financial activities,Distilling rectifying and blending of spirits,162.6115828
Auxiliary financial activities,Manufacture of wines,1208.270797
Auxiliary financial activities,Manufacture of beers,328.2755845
Auxiliary financial activities,Manufacture of soft drinks,854.2198113
Auxiliary financial activities,Manufacture of tobacco products,208.3857864
Auxiliary financial activities,Manufacture of textile products,1046.221064
Auxiliary financial activities,Manufacture of wearing apparel,1604.247807
Auxiliary financial activities,Manufacture of leather products,198.5049134
Auxiliary financial activities,Manufacture of footwear,775.2534719
Auxiliary financial activities,Sawmilling and planing of wood,356.1265777
Auxiliary financial activities,Manufacture of wood products,1323.844041
Auxiliary financial activities,Manufacture of pulp and paper,552.3517128
Auxiliary financial activities,Manufacture of containers of paper,1244.92212
Auxiliary financial activities,Manufacture of other paper products,1163.5715
Auxiliary financial activities,Printing,1379.558647
Auxiliary financial activities,Manufacture of refined petroleum products,95.15036311
Auxiliary financial activities,Manufacture of basic chemicals,1982.93005
Auxiliary financial activities,Manufacture of paints,287.2381111
Auxiliary financial activities,Manufacture of pharmaceutical products,1488.113634
Auxiliary financial activities,Manufacture of soap detergents and toilet products,1040.869161
Auxiliary financial activities,Manufacture of other chemical products,540.1548575
Auxiliary financial activities,Manufacture of rubber products,729.2266147
Auxiliary financial activities,Manufacture of plastic products,3676.621871
Auxiliary financial activities,Manufacture of glass and glass products,345.5888129
Auxiliary financial activities,Manufacture of cement,983.8028099
Auxiliary financial activities,Manufacture of concrete and concrete products,1059.59024
Auxiliary financial activities,Manufacture of basic iron and steel,2072.595487
Auxiliary financial activities,Manufacture of other basic metals,450.9253424
Auxiliary financial activities,Manufacture of fabricated metal products,4138.79333
Auxiliary financial activities,Manufacture of industrial and domestic machinery and equipment,2759.849802
Auxiliary financial activities,Manufacture of electric and electronic machinery and equipment,1960.768856
Auxiliary financial activities,Manufacture of transport equipment,858.5727302
Auxiliary financial activities,Manufacture of furniture,2521.894544
Auxiliary financial activities,Repair and installation of machinery and other manufacturing,875.0842262
Auxiliary financial activities,Electric power generation,68.91268208
Auxiliary financial activities,Transmission of electric power,7.343438329
Auxiliary financial activities,Distribution of electric power,7540.817847
Auxiliary financial activities,Gas and steam manufacture and supply,427.2707416
Auxiliary financial activities,Water collection treatment and supply,823.7548672
Auxiliary financial activities,Waste collection and recycling activities,2700.28495
Auxiliary financial activities,Construction of residential buildings,21285.14973
Auxiliary financial activities,Construction of non-residential buildings,16221.97948
Auxiliary financial activities,Civil engineering,16772.8457
Auxiliary financial activities,Specialized construction activities,10859.85348
Auxiliary financial activities,Wholesale and retail trade of motor vehicles,14088.47919
Auxiliary financial activities,Wholesale trade,69047.80238
Auxiliary financial activities,Retail trade,49356.77371
Auxiliary financial activities,Accommodation,5705.161273
Auxiliary financial activities,Food and beverage service activities,17292.31317
Auxiliary financial activities,Transport via railways,245.0761701
Auxiliary financial activities,Other passenger land transport,13203.25902
Auxiliary financial activities,Freight transport by road,5245.914014
Auxiliary financial activities,Transport via pipeline,478.6520305
Auxiliary financial activities,Water transport,669.8674188
Auxiliary financial activities,Air transport,368.7563404
Auxiliary financial activities,Warehousing,1084.110031
Auxiliary financial activities,Support activities for land transport,83.82304782
Auxiliary financial activities,Other support activities for transport,3703.754109
Auxiliary financial activities,Postal and courier activities,320.4089194
Auxiliary financial activities,Wireless telecommunications,478.8893505
Auxiliary financial activities,Wired telecommunications,242.3060862
Auxiliary financial activities,Other telecommunications activities,148.2808119
Auxiliary financial activities,Information services activities,5969.749024
Auxiliary financial activities,Publishing activities,4447.96596
Auxiliary financial activities,Banking monetary intermediation,342303.2371
Auxiliary financial activities,Insurance activities,288223.3985
Auxiliary financial activities,Auxiliary financial activities,91570.25577
Auxiliary financial activities,Real estate activities,22798.89494
Auxiliary financial activities,Housing services,0
Auxiliary financial activities,Legal and accounting services,12848.19608
Auxiliary financial activities,Architectural and engineering services,9223.197102
Auxiliary financial activities,Other professional activities,18645.32608
Auxiliary financial activities,Rental and leasing activities,28093.89383
Auxiliary financial activities,Business support activities,2399.618722
Auxiliary financial activities,Public administration,625.951182
Auxiliary financial activities,Public education,539.0196626
Auxiliary financial activities,Private education,2855.51925
Auxiliary financial activities,Public health activities,188.6270979
Auxiliary financial activities,Private health activities,373.7795303
Auxiliary financial activities,Activities of organizations,844.2833258
Auxiliary financial activities,Artistic and entertainment activities,5794.598907
Auxiliary financial activities,Other personal services,1163.131676
Real estate activities,Cultivation of annual crops,5653.220734
Real estate activities,Cultivation of vegetables,2920.702747
Real estate activities,Cultivation of grapes,4009.80292
Real estate activities,Cultivation of other fruits,12087.78684
Real estate activities,Cattle bredding,4673.023114
Real estate activities,Pigs breeding,2091.157961
Real estate activities,Chicken breeding,3627.430211
Real estate activities,Breeding of other animals,377.2026008
Real estate activities,Farming services,263.6180633
Real estate activities,Forestry,322.3125584
Real estate activities,Aquaculture,13103.2712
Real estate activities,Extractive fishing,46.2558205
Real estate activities,Coal mining,10.58258892
Real estate activities,Crude oil and gas mining,260.4727371
Real estate activities,Copper mining,55909.5955
Real estate activities,Iron mining,16407.35146
Real estate activities,Mining of other metals,928.7546647
Real estate activities,Other mining activities and mining services,241.5048349
Real estate activities,Processing and preserving of meat,6405.937559
Real estate activities,Processing and preserving of fishmeal and fish oil,9.850204503
Real estate activities,Processing and preserving of fish,1497.08878
Real estate activities,Processing and preserving of fruits and vegetables,4361.035236
Real estate activities,Manufacture of vegetable and animal oils,629.7509645
Real estate activities,Manufacture of dairy products,6441.757402
Real estate activities,Manufacture of grain mill products,1879.92168
Real estate activities,Manufacture of prepared animal feeds,3234.804639
Real estate activities,Manufacture of bakery products,20645.37425
Real estate activities,Manufacture of macaroni noodles and other related products,3012.577153
Real estate activities,Manufacture of other food products,5527.440632
Real estate activities,Distilling rectifying and blending of spirits,1262.488995
Real estate activities,Manufacture of wines,7429.481891
Real estate activities,Manufacture of beers,1265.307507
Real estate activities,Manufacture of soft drinks,4426.561332
Real estate activities,Manufacture of tobacco products,38.03211365
Real estate activities,Manufacture of textile products,7338.577584
Real estate activities,Manufacture of wearing apparel,20498.05382
Real estate activities,Manufacture of leather products,898.4324219
Real estate activities,Manufacture of footwear,5035.021187
Real estate activities,Sawmilling and planing of wood,5758.60075
Real estate activities,Manufacture of wood products,4041.652104
Real estate activities,Manufacture of pulp and paper,1085.085526
Real estate activities,Manufacture of containers of paper,3015.28175
Real estate activities,Manufacture of other paper products,8288.168616
Real estate activities,Printing,8969.599534
Real estate activities,Manufacture of refined petroleum products,1469.827399
Real estate activities,Manufacture of basic chemicals,4382.426073
Real estate activities,Manufacture of paints,3049.492523
Real estate activities,Manufacture of pharmaceutical products,8236.089678
Real estate activities,Manufacture of soap detergents and toilet products,2613.879507
Real estate activities,Manufacture of other chemical products,7125.973202
Real estate activities,Manufacture of rubber products,1136.747292
Real estate activities,Manufacture of plastic products,13483.05931
Real estate activities,Manufacture of glass and glass products,2301.038359
Real estate activities,Manufacture of cement,3278.844581
Real estate activities,Manufacture of concrete and concrete products,12158.65891
Real estate activities,Manufacture of basic iron and steel,2022.594677
Real estate activities,Manufacture of other basic metals,222.5442371
Real estate activities,Manufacture of fabricated metal products,16221.2093
Real estate activities,Manufacture of industrial and domestic machinery and equipment,10518.67996
Real estate activities,Manufacture of electric and electronic machinery and equipment,10294.02509
Real estate activities,Manufacture of transport equipment,4598.840457
Real estate activities,Manufacture of furniture,13698.09959
Real estate activities,Repair and installation of machinery and other manufacturing,28123.82925
Real estate activities,Electric power generation,10893.67773
Real estate activities,Transmission of electric power,2291.87841
Real estate activities,Distribution of electric power,8396.332265
Real estate activities,Gas and steam manufacture and supply,1931.228201
Real estate activities,Water collection treatment and supply,5045.37178
Real estate activities,Waste collection and recycling activities,6539.963795
Real estate activities,Construction of residential buildings,34511.52479
Real estate activities,Construction of non-residential buildings,15583.18951
Real estate activities,Civil engineering,22250.95205
Real estate activities,Specialized construction activities,4842.59868
Real estate activities,Wholesale and retail trade of motor vehicles,100626.6171
Real estate activities,Wholesale trade,360074.9672
Real estate activities,Retail trade,1097226.459
Real estate activities,Accommodation,110205.5268
Real estate activities,Food and beverage service activities,190611.2255
Real estate activities,Transport via railways,1231.573274
Real estate activities,Other passenger land transport,20721.70069
Real estate activities,Freight transport by road,59387.02856
Real estate activities,Transport via pipeline,962.5500011
Real estate activities,Water transport,747.4422826
Real estate activities,Air transport,75972.9124
Real estate activities,Warehousing,41775.30361
Real estate activities,Support activities for land transport,18661.03874
Real estate activities,Other support activities for transport,44237.46222
Real estate activities,Postal and courier activities,3264.057541
Real estate activities,Wireless telecommunications,157384.9556
Real estate activities,Wired telecommunications,26316.09878
Real estate activities,Other telecommunications activities,8342.48708
Real estate activities,Information services activities,70491.67099
Real estate activities,Publishing activities,55123.32456
Real estate activities,Banking monetary intermediation,103097.1043
Real estate activities,Insurance activities,24792.11278
Real estate activities,Auxiliary financial activities,13983.76133
Real estate activities,Real estate activities,254320.969
Real estate activities,Housing services,0
Real estate activities,Legal and accounting services,72902.21504
Real estate activities,Architectural and engineering services,164530.896
Real estate activities,Other professional activities,154122.0974
Real estate activities,Rental and leasing activities,53814.29854
Real estate activities,Business support activities,158056.8631
Real estate activities,Public administration,91604.19792
Real estate activities,Public education,203275.0204
Real estate activities,Private education,125918.434
Real estate activities,Public health activities,17899.6551
Real estate activities,Private health activities,118325.8115
Real estate activities,Activities of organizations,35611.03246
Real estate activities,Artistic and entertainment activities,85284.06722
Real estate activities,Other personal services,35876.36432
Housing services,Cultivation of annual crops,0
Housing services,Cultivation of vegetables,0
Housing services,Cultivation of grapes,0
Housing services,Cultivation of other fruits,0
Housing services,Cattle bredding,0
Housing services,Pigs breeding,0
Housing services,Chicken breeding,0
Housing services,Breeding of other animals,0
Housing services,Farming services,0
Housing services,Forestry,0
Housing services,Aquaculture,0
Housing services,Extractive fishing,0
Housing services,Coal mining,0
Housing services,Crude oil and gas mining,0
Housing services,Copper mining,0
Housing services,Iron mining,0
Housing services,Mining of other metals,0
Housing services,Other mining activities and mining services,0
Housing services,Processing and preserving of meat,0
Housing services,Processing and preserving of fishmeal and fish oil,0
Housing services,Processing and preserving of fish,0
Housing services,Processing and preserving of fruits and vegetables,0
Housing services,Manufacture of vegetable and animal oils,0
Housing services,Manufacture of dairy products,0
Housing services,Manufacture of grain mill products,0
Housing services,Manufacture of prepared animal feeds,0
Housing services,Manufacture of bakery products,0
Housing services,Manufacture of macaroni noodles and other related products,0
Housing services,Manufacture of other food products,0
Housing services,Distilling rectifying and blending of spirits,0
Housing services,Manufacture of wines,0
Housing services,Manufacture of beers,0
Housing services,Manufacture of soft drinks,0
Housing services,Manufacture of tobacco products,0
Housing services,Manufacture of textile products,0
Housing services,Manufacture of wearing apparel,0
Housing services,Manufacture of leather products,0
Housing services,Manufacture of footwear,0
Housing services,Sawmilling and planing of wood,0
Housing services,Manufacture of wood products,0
Housing services,Manufacture of pulp and paper,0
Housing services,Manufacture of containers of paper,0
Housing services,Manufacture of other paper products,0
Housing services,Printing,0
Housing services,Manufacture of refined petroleum products,0
Housing services,Manufacture of basic chemicals,0
Housing services,Manufacture of paints,0
Housing services,Manufacture of pharmaceutical products,0
Housing services,Manufacture of soap detergents and toilet products,0
Housing services,Manufacture of other chemical products,0
Housing services,Manufacture of rubber products,0
Housing services,Manufacture of plastic products,0
Housing services,Manufacture of glass and glass products,0
Housing services,Manufacture of cement,0
Housing services,Manufacture of concrete and concrete products,0
Housing services,Manufacture of basic iron and steel,0
Housing services,Manufacture of other basic metals,0
Housing services,Manufacture of fabricated metal products,0
Housing services,Manufacture of industrial and domestic machinery and equipment,0
Housing services,Manufacture of electric and electronic machinery and equipment,0
Housing services,Manufacture of transport equipment,0
Housing services,Manufacture of furniture,0
Housing services,Repair and installation of machinery and other manufacturing,0
Housing services,Electric power generation,0
Housing services,Transmission of electric power,0
Housing services,Distribution of electric power,0
Housing services,Gas and steam manufacture and supply,0
Housing services,Water collection treatment and supply,0
Housing services,Waste collection and recycling activities,0
Housing services,Construction of residential buildings,0
Housing services,Construction of non-residential buildings,0
Housing services,Civil engineering,0
Housing services,Specialized construction activities,0
Housing services,Wholesale and retail trade of motor vehicles,0
Housing services,Wholesale trade,0
Housing services,Retail trade,0
Housing services,Accommodation,0
Housing services,Food and beverage service activities,0
Housing services,Transport via railways,0
Housing services,Other passenger land transport,0
Housing services,Freight transport by road,0
Housing services,Transport via pipeline,0
Housing services,Water transport,0
Housing services,Air transport,0
Housing services,Warehousing,0
Housing services,Support activities for land transport,0
Housing services,Other support activities for transport,0
Housing services,Postal and courier activities,0
Housing services,Wireless telecommunications,0
Housing services,Wired telecommunications,0
Housing services,Other telecommunications activities,0
Housing services,Information services activities,0
Housing services,Publishing activities,0
Housing services,Banking monetary intermediation,0
Housing services,Insurance activities,0
Housing services,Auxiliary financial activities,0
Housing services,Real estate activities,0
Housing services,Housing services,0
Housing services,Legal and accounting services,0
Housing services,Architectural and engineering services,0
Housing services,Other professional activities,0
Housing services,Rental and leasing activities,0
Housing services,Business support activities,0
Housing services,Public administration,0
Housing services,Public education,0
Housing services,Private education,0
Housing services,Public health activities,0
Housing services,Private health activities,0
Housing services,Activities of organizations,0
Housing services,Artistic and entertainment activities,0
Housing services,Other personal services,0
Legal and accounting services,Cultivation of annual crops,2176.970637
Legal and accounting services,Cultivation of vegetables,1509.266532
Legal and accounting services,Cultivation of grapes,1447.263659
Legal and accounting services,Cultivation of other fruits,3688.128397
Legal and accounting services,Cattle bredding,1814.854975
Legal and accounting services,Pigs breeding,879.6044075
Legal and accounting services,Chicken breeding,1840.636173
Legal and accounting services,Breeding of other animals,158.8074209
Legal and accounting services,Farming services,840.9368682
Legal and accounting services,Forestry,91.18334201
Legal and accounting services,Aquaculture,13550.54845
Legal and accounting services,Extractive fishing,0.228432847
Legal and accounting services,Coal mining,4.449058443
Legal and accounting services,Crude oil and gas mining,67.34768217
Legal and accounting services,Copper mining,82226.17386
Legal and accounting services,Iron mining,11572.1609
Legal and accounting services,Mining of other metals,2702.982984
Legal and accounting services,Other mining activities and mining services,109.7258972
Legal and accounting services,Processing and preserving of meat,6327.093377
Legal and accounting services,Processing and preserving of fishmeal and fish oil,3746.306775
Legal and accounting services,Processing and preserving of fish,11111.45045
Legal and accounting services,Processing and preserving of fruits and vegetables,5886.829508
Legal and accounting services,Manufacture of vegetable and animal oils,3008.964872
Legal and accounting services,Manufacture of dairy products,17081.66814
Legal and accounting services,Manufacture of grain mill products,3926.209178
Legal and accounting services,Manufacture of prepared animal feeds,17369.19695
Legal and accounting services,Manufacture of bakery products,12739.31234
Legal and accounting services,Manufacture of macaroni noodles and other related products,1495.135463
Legal and accounting services,Manufacture of other food products,8675.825137
Legal and accounting services,Distilling rectifying and blending of spirits,2561.067591
Legal and accounting services,Manufacture of wines,15061.47857
Legal and accounting services,Manufacture of beers,7340.520123
Legal and accounting services,Manufacture of soft drinks,19936.29389
Legal and accounting services,Manufacture of tobacco products,2558.600417
Legal and accounting services,Manufacture of textile products,4212.880865
Legal and accounting services,Manufacture of wearing apparel,7214.936866
Legal and accounting services,Manufacture of leather products,541.9688489
Legal and accounting services,Manufacture of footwear,1964.849841
Legal and accounting services,Sawmilling and planing of wood,17037.7455
Legal and accounting services,Manufacture of wood products,16326.98494
Legal and accounting services,Manufacture of pulp and paper,37288.97972
Legal and accounting services,Manufacture of containers of paper,4522.634156
Legal and accounting services,Manufacture of other paper products,4935.019812
Legal and accounting services,Printing,11925.65739
Legal and accounting services,Manufacture of refined petroleum products,10595.44698
Legal and accounting services,Manufacture of basic chemicals,23374.30499
Legal and accounting services,Manufacture of paints,2757.332823
Legal and accounting services,Manufacture of pharmaceutical products,16408.54876
Legal and accounting services,Manufacture of soap detergents and toilet products,5977.616293
Legal and accounting services,Manufacture of other chemical products,2145.772324
Legal and accounting services,Manufacture of rubber products,1777.843168
Legal and accounting services,Manufacture of plastic products,15501.13303
Legal and accounting services,Manufacture of glass and glass products,3775.138483
Legal and accounting services,Manufacture of cement,6516.623425
Legal and accounting services,Manufacture of concrete and concrete products,18166.06073
Legal and accounting services,Manufacture of basic iron and steel,37395.94144
Legal and accounting services,Manufacture of other basic metals,3528.23351
Legal and accounting services,Manufacture of fabricated metal products,37659.45109
Legal and accounting services,Manufacture of industrial and domestic machinery and equipment,20322.59744
Legal and accounting services,Manufacture of electric and electronic machinery and equipment,15439.73041
Legal and accounting services,Manufacture of transport equipment,4728.546449
Legal and accounting services,Manufacture of furniture,5985.427779
Legal and accounting services,Repair and installation of machinery and other manufacturing,11446.71472
Legal and accounting services,Electric power generation,49722.33155
Legal and accounting services,Transmission of electric power,4577.416413
Legal and accounting services,Distribution of electric power,13833.86329
Legal and accounting services,Gas and steam manufacture and supply,7334.755258
Legal and accounting services,Water collection treatment and supply,23892.54306
Legal and accounting services,Waste collection and recycling activities,16865.72195
Legal and accounting services,Construction of residential buildings,24325.9837
Legal and accounting services,Construction of non-residential buildings,8920.073155
Legal and accounting services,Civil engineering,4809.603681
Legal and accounting services,Specialized construction activities,994.4507279
Legal and accounting services,Wholesale and retail trade of motor vehicles,39248.92037
Legal and accounting services,Wholesale trade,145384.6976
Legal and accounting services,Retail trade,133026.0549
Legal and accounting services,Accommodation,20955.9836
Legal and accounting services,Food and beverage service activities,36514.18713
Legal and accounting services,Transport via railways,187.2978019
Legal and accounting services,Other passenger land transport,4351.785534
Legal and accounting services,Freight transport by road,2902.088051
Legal and accounting services,Transport via pipeline,167.8188815
Legal and accounting services,Water transport,1525.114017
Legal and accounting services,Air transport,83635.48996
Legal and accounting services,Warehousing,8217.874176
Legal and accounting services,Support activities for land transport,12080.64339
Legal and accounting services,Other support activities for transport,16636.17854
Legal and accounting services,Postal and courier activities,3611.988888
Legal and accounting services,Wireless telecommunications,7753.104193
Legal and accounting services,Wired telecommunications,8296.401508
Legal and accounting services,Other telecommunications activities,13590.6395
Legal and accounting services,Information services activities,38975.53438
Legal and accounting services,Publishing activities,28648.42711
Legal and accounting services,Banking monetary intermediation,76271.84964
Legal and accounting services,Insurance activities,71075.85274
Legal and accounting services,Auxiliary financial activities,15137.25486
Legal and accounting services,Real estate activities,55018.28564
Legal and accounting services,Housing services,0
Legal and accounting services,Legal and accounting services,83755.34189
Legal and accounting services,Architectural and engineering services,77045.94499
Legal and accounting services,Other professional activities,130340.1036
Legal and accounting services,Rental and leasing activities,16961.5038
Legal and accounting services,Business support activities,132043.8143
Legal and accounting services,Public administration,19944.47435
Legal and accounting services,Public education,59514.45699
Legal and accounting services,Private education,16875.22339
Legal and accounting services,Public health activities,2364.92
Legal and accounting services,Private health activities,30812.24623
Legal and accounting services,Activities of organizations,34061.35366
Legal and accounting services,Artistic and entertainment activities,56725.42344
Legal and accounting services,Other personal services,16337.92442
Architectural and engineering services,Cultivation of annual crops,142.6602136
Architectural and engineering services,Cultivation of vegetables,53.75467272
Architectural and engineering services,Cultivation of grapes,192.0480038
Architectural and engineering services,Cultivation of other fruits,252.1832717
Architectural and engineering services,Cattle bredding,178.5257049
Architectural and engineering services,Pigs breeding,119.6550953
Architectural and engineering services,Chicken breeding,103.9875415
Architectural and engineering services,Breeding of other animals,44.11023104
Architectural and engineering services,Farming services,3156.213967
Architectural and engineering services,Forestry,216.9303399
Architectural and engineering services,Aquaculture,360.529227
Architectural and engineering services,Extractive fishing,2.701885103
Architectural and engineering services,Coal mining,1513.928518
Architectural and engineering services,Crude oil and gas mining,7.469437856
Architectural and engineering services,Copper mining,1288810.205
Architectural and engineering services,Iron mining,150199.118
Architectural and engineering services,Mining of other metals,124674.0371
Architectural and engineering services,Other mining activities and mining services,93163.14133
Architectural and engineering services,Processing and preserving of meat,9421.491447
Architectural and engineering services,Processing and preserving of fishmeal and fish oil,570.6308376
Architectural and engineering services,Processing and preserving of fish,10538.28174
Architectural and engineering services,Processing and preserving of fruits and vegetables,804.7450439
Architectural and engineering services,Manufacture of vegetable and animal oils,1570.807105
Architectural and engineering services,Manufacture of dairy products,1228.219671
Architectural and engineering services,Manufacture of grain mill products,8296.494787
Architectural and engineering services,Manufacture of prepared animal feeds,2880.360331
Architectural and engineering services,Manufacture of bakery products,5609.345105
Architectural and engineering services,Manufacture of macaroni noodles and other related products,18.37552124
Architectural and engineering services,Manufacture of other food products,7915.724773
Architectural and engineering services,Distilling rectifying and blending of spirits,14595.20678
Architectural and engineering services,Manufacture of wines,16302.59833
Architectural and engineering services,Manufacture of beers,5189.29014
Architectural and engineering services,Manufacture of soft drinks,2014.110785
Architectural and engineering services,Manufacture of tobacco products,7744.896958
Architectural and engineering services,Manufacture of textile products,3098.977159
Architectural and engineering services,Manufacture of wearing apparel,10760.89814
Architectural and engineering services,Manufacture of leather products,46.0499609
Architectural and engineering services,Manufacture of footwear,1204.915165
Architectural and engineering services,Sawmilling and planing of wood,7157.388289
Architectural and engineering services,Manufacture of wood products,3012.411698
Architectural and engineering services,Manufacture of pulp and paper,7494.117508
Architectural and engineering services,Manufacture of containers of paper,670.6471849
Architectural and engineering services,Manufacture of other paper products,5915.331714
Architectural and engineering services,Printing,13051.9454
Architectural and engineering services,Manufacture of refined petroleum products,2727.643699
Architectural and engineering services,Manufacture of basic chemicals,28709.59755
Architectural and engineering services,Manufacture of paints,1095.740418
Architectural and engineering services,Manufacture of pharmaceutical products,16596.67129
Architectural and engineering services,Manufacture of soap detergents and toilet products,1198.35239
Architectural and engineering services,Manufacture of other chemical products,4899.711838
Architectural and engineering services,Manufacture of rubber products,461.1966639
Architectural and engineering services,Manufacture of plastic products,1913.914737
Architectural and engineering services,Manufacture of glass and glass products,169.6704306
Architectural and engineering services,Manufacture of cement,9341.420828
Architectural and engineering services,Manufacture of concrete and concrete products,5357.443779
Architectural and engineering services,Manufacture of basic iron and steel,592.3459197
Architectural and engineering services,Manufacture of other basic metals,1578.243377
Architectural and engineering services,Manufacture of fabricated metal products,21551.62995
Architectural and engineering services,Manufacture of industrial and domestic machinery and equipment,19307.44616
Architectural and engineering services,Manufacture of electric and electronic machinery and equipment,2549.911206
Architectural and engineering services,Manufacture of transport equipment,4431.863965
Architectural and engineering services,Manufacture of furniture,15489.47354
Architectural and engineering services,Repair and installation of machinery and other manufacturing,963.8356517
Architectural and engineering services,Electric power generation,16246.20397
Architectural and engineering services,Transmission of electric power,1110.697879
Architectural and engineering services,Distribution of electric power,1136.054052
Architectural and engineering services,Gas and steam manufacture and supply,2490.364216
Architectural and engineering services,Water collection treatment and supply,16604.3218
Architectural and engineering services,Waste collection and recycling activities,2510.529899
Architectural and engineering services,Construction of residential buildings,42534.95563
Architectural and engineering services,Construction of non-residential buildings,22041.02075
Architectural and engineering services,Civil engineering,330340.3445
Architectural and engineering services,Specialized construction activities,149727.998
Architectural and engineering services,Wholesale and retail trade of motor vehicles,14386.5214
Architectural and engineering services,Wholesale trade,221629.9421
Architectural and engineering services,Retail trade,39895.63358
Architectural and engineering services,Accommodation,5409.010278
Architectural and engineering services,Food and beverage service activities,2115.703954
Architectural and engineering services,Transport via railways,231.0697624
Architectural and engineering services,Other passenger land transport,21690.51613
Architectural and engineering services,Freight transport by road,39156.97788
Architectural and engineering services,Transport via pipeline,568.8242095
Architectural and engineering services,Water transport,1373.288116
Architectural and engineering services,Air transport,3105.569019
Architectural and engineering services,Warehousing,300.8472692
Architectural and engineering services,Support activities for land transport,2140.523389
Architectural and engineering services,Other support activities for transport,12814.25203
Architectural and engineering services,Postal and courier activities,2154.664149
Architectural and engineering services,Wireless telecommunications,1521.164474
Architectural and engineering services,Wired telecommunications,816.7861184
Architectural and engineering services,Other telecommunications activities,8622.484606
Architectural and engineering services,Information services activities,6418.303937
Architectural and engineering services,Publishing activities,7855.772788
Architectural and engineering services,Banking monetary intermediation,6456.349454
Architectural and engineering services,Insurance activities,17140.44722
Architectural and engineering services,Auxiliary financial activities,1567.631177
Architectural and engineering services,Real estate activities,38784.58759
Architectural and engineering services,Housing services,0
Architectural and engineering services,Legal and accounting services,5418.454816
Architectural and engineering services,Architectural and engineering services,724131.992
Architectural and engineering services,Other professional activities,75361.38185
Architectural and engineering services,Rental and leasing activities,16617.33925
Architectural and engineering services,Business support activities,29083.01965
Architectural and engineering services,Public administration,22345.84987
Architectural and engineering services,Public education,1439.668134
Architectural and engineering services,Private education,2856.990935
Architectural and engineering services,Public health activities,2611.028827
Architectural and engineering services,Private health activities,2334.480002
Architectural and engineering services,Activities of organizations,2018.551356
Architectural and engineering services,Artistic and entertainment activities,5906.417584
Architectural and engineering services,Other personal services,5846.827198
Other professional activities,Cultivation of annual crops,5202.469728
Other professional activities,Cultivation of vegetables,2052.907985
Other professional activities,Cultivation of grapes,2638.529691
Other professional activities,Cultivation of other fruits,7039.792636
Other professional activities,Cattle bredding,6378.917137
Other professional activities,Pigs breeding,2620.380356
Other professional activities,Chicken breeding,5135.125782
Other professional activities,Breeding of other animals,619.0520717
Other professional activities,Farming services,437.2921097
Other professional activities,Forestry,1131.906322
Other professional activities,Aquaculture,5039.688276
Other professional activities,Extractive fishing,79.52695966
Other professional activities,Coal mining,93.75455425
Other professional activities,Crude oil and gas mining,222.0718961
Other professional activities,Copper mining,159835.0792
Other professional activities,Iron mining,2284.298419
Other professional activities,Mining of other metals,1564.228418
Other professional activities,Other mining activities and mining services,2018.325417
Other professional activities,Processing and preserving of meat,91444.12252
Other professional activities,Processing and preserving of fishmeal and fish oil,4447.831009
Other professional activities,Processing and preserving of fish,62626.56479
Other professional activities,Processing and preserving of fruits and vegetables,32265.93448
Other professional activities,Manufacture of vegetable and animal oils,7785.544922
Other professional activities,Manufacture of dairy products,46265.48748
Other professional activities,Manufacture of grain mill products,27331.6152
Other professional activities,Manufacture of prepared animal feeds,36658.73416
Other professional activities,Manufacture of bakery products,34522.41176
Other professional activities,Manufacture of macaroni noodles and other related products,897.6985111
Other professional activities,Manufacture of other food products,43441.01595
Other professional activities,Distilling rectifying and blending of spirits,13187.69925
Other professional activities,Manufacture of wines,88016.66866
Other professional activities,Manufacture of beers,41515.57365
Other professional activities,Manufacture of soft drinks,78039.12553
Other professional activities,Manufacture of tobacco products,11311.92837
Other professional activities,Manufacture of textile products,13481.43573
Other professional activities,Manufacture of wearing apparel,37401.0841
Other professional activities,Manufacture of leather products,1051.028152
Other professional activities,Manufacture of footwear,3379.075918
Other professional activities,Sawmilling and planing of wood,55265.03473
Other professional activities,Manufacture of wood products,52727.9564
Other professional activities,Manufacture of pulp and paper,52675.74332
Other professional activities,Manufacture of containers of paper,18837.73276
Other professional activities,Manufacture of other paper products,36840.26411
Other professional activities,Printing,28228.82981
Other professional activities,Manufacture of refined petroleum products,36211.35437
Other professional activities,Manufacture of basic chemicals,40543.84946
Other professional activities,Manufacture of paints,13845.67136
Other professional activities,Manufacture of pharmaceutical products,115255.3052
Other professional activities,Manufacture of soap detergents and toilet products,45024.61033
Other professional activities,Manufacture of other chemical products,26083.11758
Other professional activities,Manufacture of rubber products,726.3257861
Other professional activities,Manufacture of plastic products,26849.17862
Other professional activities,Manufacture of glass and glass products,6230.953049
Other professional activities,Manufacture of cement,23450.24655
Other professional activities,Manufacture of concrete and concrete products,65161.23751
Other professional activities,Manufacture of basic iron and steel,17257.40187
Other professional activities,Manufacture of other basic metals,6956.967634
Other professional activities,Manufacture of fabricated metal products,79593.38773
Other professional activities,Manufacture of industrial and domestic machinery and equipment,38546.20647
Other professional activities,Manufacture of electric and electronic machinery and equipment,12966.20318
Other professional activities,Manufacture of transport equipment,24297.28273
Other professional activities,Manufacture of furniture,20479.13107
Other professional activities,Repair and installation of machinery and other manufacturing,17240.75055
Other professional activities,Electric power generation,33994.81826
Other professional activities,Transmission of electric power,616.0841494
Other professional activities,Distribution of electric power,4500.961763
Other professional activities,Gas and steam manufacture and supply,90802.24711
Other professional activities,Water collection treatment and supply,7099.517636
Other professional activities,Waste collection and recycling activities,3850.320575
Other professional activities,Construction of residential buildings,90740.27288
Other professional activities,Construction of non-residential buildings,31980.16439
Other professional activities,Civil engineering,5024.80791
Other professional activities,Specialized construction activities,3508.438534
Other professional activities,Wholesale and retail trade of motor vehicles,76599.10036
Other professional activities,Wholesale trade,486816.1321
Other professional activities,Retail trade,324782.858
Other professional activities,Accommodation,8924.10536
Other professional activities,Food and beverage service activities,32667.2028
Other professional activities,Transport via railways,5435.56485
Other professional activities,Other passenger land transport,13875.05096
Other professional activities,Freight transport by road,17665.03309
Other professional activities,Transport via pipeline,60.9843349
Other professional activities,Water transport,3036.733918
Other professional activities,Air transport,36859.71356
Other professional activities,Warehousing,14372.0382
Other professional activities,Support activities for land transport,23832.73526
Other professional activities,Other support activities for transport,9730.233624
Other professional activities,Postal and courier activities,2051.613374
Other professional activities,Wireless telecommunications,68270.04711
Other professional activities,Wired telecommunications,19580.19275
Other professional activities,Other telecommunications activities,80888.06596
Other professional activities,Information services activities,110517.9548
Other professional activities,Publishing activities,56705.06311
Other professional activities,Banking monetary intermediation,213623.4634
Other professional activities,Insurance activities,132733.9536
Other professional activities,Auxiliary financial activities,59693.52077
Other professional activities,Real estate activities,119138.268
Other professional activities,Housing services,0
Other professional activities,Legal and accounting services,86323.74627
Other professional activities,Architectural and engineering services,169311.7873
Other professional activities,Other professional activities,351617.9341
Other professional activities,Rental and leasing activities,53944.30682
Other professional activities,Business support activities,267096.8737
Other professional activities,Public administration,94992.99106
Other professional activities,Public education,61262.49166
Other professional activities,Private education,118798.4287
Other professional activities,Public health activities,43976.0005
Other professional activities,Private health activities,74987.99919
Other professional activities,Activities of organizations,4100.762303
Other professional activities,Artistic and entertainment activities,33337.44731
Other professional activities,Other personal services,3316.790606
Rental and leasing activities,Cultivation of annual crops,716.1032663
Rental and leasing activities,Cultivation of vegetables,313.3378422
Rental and leasing activities,Cultivation of grapes,564.8397611
Rental and leasing activities,Cultivation of other fruits,1134.65741
Rental and leasing activities,Cattle bredding,179.4639511
Rental and leasing activities,Pigs breeding,2.711860811
Rental and leasing activities,Chicken breeding,45.81072357
Rental and leasing activities,Breeding of other animals,16.92187476
Rental and leasing activities,Farming services,159.4447505
Rental and leasing activities,Forestry,87928.9652
Rental and leasing activities,Aquaculture,39943.75548
Rental and leasing activities,Extractive fishing,168.463246
Rental and leasing activities,Coal mining,246.5594677
Rental and leasing activities,Crude oil and gas mining,7091.392935
Rental and leasing activities,Copper mining,190577.5519
Rental and leasing activities,Iron mining,7568.029569
Rental and leasing activities,Mining of other metals,5350.572504
Rental and leasing activities,Other mining activities and mining services,8428.824462
Rental and leasing activities,Processing and preserving of meat,2826.042656
Rental and leasing activities,Processing and preserving of fishmeal and fish oil,2.182781226
Rental and leasing activities,Processing and preserving of fish,1578.344889
Rental and leasing activities,Processing and preserving of fruits and vegetables,2924.299171
Rental and leasing activities,Manufacture of vegetable and animal oils,503.6270484
Rental and leasing activities,Manufacture of dairy products,74.91153839
Rental and leasing activities,Manufacture of grain mill products,26.62503517
Rental and leasing activities,Manufacture of prepared animal feeds,5820.232495
Rental and leasing activities,Manufacture of bakery products,1363.801135
Rental and leasing activities,Manufacture of macaroni noodles and other related products,289.0211482
Rental and leasing activities,Manufacture of other food products,1957.326365
Rental and leasing activities,Distilling rectifying and blending of spirits,467.1500521
Rental and leasing activities,Manufacture of wines,6473.656585
Rental and leasing activities,Manufacture of beers,15.4612281
Rental and leasing activities,Manufacture of soft drinks,761.6714976
Rental and leasing activities,Manufacture of tobacco products,2342.381206
Rental and leasing activities,Manufacture of textile products,864.5537161
Rental and leasing activities,Manufacture of wearing apparel,71.8118315
Rental and leasing activities,Manufacture of leather products,21.11513235
Rental and leasing activities,Manufacture of footwear,64.975202
Rental and leasing activities,Sawmilling and planing of wood,1736.678749
Rental and leasing activities,Manufacture of wood products,1163.473982
Rental and leasing activities,Manufacture of pulp and paper,395.670063
Rental and leasing activities,Manufacture of containers of paper,1241.751437
Rental and leasing activities,Manufacture of other paper products,44.03442103
Rental and leasing activities,Printing,2024.6201
Rental and leasing activities,Manufacture of refined petroleum products,47.12233235
Rental and leasing activities,Manufacture of basic chemicals,11600.63474
Rental and leasing activities,Manufacture of paints,13.17514763
Rental and leasing activities,Manufacture of pharmaceutical products,276.4432315
Rental and leasing activities,Manufacture of soap detergents and toilet products,57.2002667
Rental and leasing activities,Manufacture of other chemical products,6.172799155
Rental and leasing activities,Manufacture of rubber products,3.403963211
Rental and leasing activities,Manufacture of plastic products,1462.514002
Rental and leasing activities,Manufacture of glass and glass products,12.62394229
Rental and leasing activities,Manufacture of cement,2698.892617
Rental and leasing activities,Manufacture of concrete and concrete products,17177.61206
Rental and leasing activities,Manufacture of basic iron and steel,3358.560348
Rental and leasing activities,Manufacture of other basic metals,132.6435915
Rental and leasing activities,Manufacture of fabricated metal products,8294.928162
Rental and leasing activities,Manufacture of industrial and domestic machinery and equipment,2387.514077
Rental and leasing activities,Manufacture of electric and electronic machinery and equipment,1267.446516
Rental and leasing activities,Manufacture of transport equipment,516.2881706
Rental and leasing activities,Manufacture of furniture,736.5108835
Rental and leasing activities,Repair and installation of machinery and other manufacturing,11820.23683
Rental and leasing activities,Electric power generation,5236.364811
Rental and leasing activities,Transmission of electric power,615.255222
Rental and leasing activities,Distribution of electric power,12266.93652
Rental and leasing activities,Gas and steam manufacture and supply,7837.465624
Rental and leasing activities,Water collection treatment and supply,6758.692675
Rental and leasing activities,Waste collection and recycling activities,5816.153335
Rental and leasing activities,Construction of residential buildings,96873.7938
Rental and leasing activities,Construction of non-residential buildings,52458.04414
Rental and leasing activities,Civil engineering,319442.9735
Rental and leasing activities,Specialized construction activities,17355.98237
Rental and leasing activities,Wholesale and retail trade of motor vehicles,14740.36862
Rental and leasing activities,Wholesale trade,95233.21722
Rental and leasing activities,Retail trade,56168.89747
Rental and leasing activities,Accommodation,5657.004019
Rental and leasing activities,Food and beverage service activities,32455.08118
Rental and leasing activities,Transport via railways,469.8706254
Rental and leasing activities,Other passenger land transport,18477.48614
Rental and leasing activities,Freight transport by road,213549.7196
Rental and leasing activities,Transport via pipeline,29.72353675
Rental and leasing activities,Water transport,19024.69472
Rental and leasing activities,Air transport,36185.64335
Rental and leasing activities,Warehousing,3216.427565
Rental and leasing activities,Support activities for land transport,8769.898965
Rental and leasing activities,Other support activities for transport,55366.02894
Rental and leasing activities,Postal and courier activities,1583.399235
Rental and leasing activities,Wireless telecommunications,6667.397582
Rental and leasing activities,Wired telecommunications,11575.05997
Rental and leasing activities,Other telecommunications activities,29088.08502
Rental and leasing activities,Information services activities,24572.79536
Rental and leasing activities,Publishing activities,19655.91316
Rental and leasing activities,Banking monetary intermediation,561.9958771
Rental and leasing activities,Insurance activities,3950.616466
Rental and leasing activities,Auxiliary financial activities,130.5065662
Rental and leasing activities,Real estate activities,12680.49711
Rental and leasing activities,Housing services,0
Rental and leasing activities,Legal and accounting services,38272.61492
Rental and leasing activities,Architectural and engineering services,57796.96867
Rental and leasing activities,Other professional activities,43260.34433
Rental and leasing activities,Rental and leasing activities,77253.06121
Rental and leasing activities,Business support activities,52002.3949
Rental and leasing activities,Public administration,82375.16455
Rental and leasing activities,Public education,7538.608507
Rental and leasing activities,Private education,2263.021894
Rental and leasing activities,Public health activities,17021.67149
Rental and leasing activities,Private health activities,25301.20162
Rental and leasing activities,Activities of organizations,883.7110477
Rental and leasing activities,Artistic and entertainment activities,42503.24652
Rental and leasing activities,Other personal services,4575.774738
Business support activities,Cultivation of annual crops,12611.86114
Business support activities,Cultivation of vegetables,10585.60748
Business support activities,Cultivation of grapes,2259.159625
Business support activities,Cultivation of other fruits,6547.125229
Business support activities,Cattle bredding,24552.48129
Business support activities,Pigs breeding,11611.84778
Business support activities,Chicken breeding,31272.64261
Business support activities,Breeding of other animals,2272.399935
Business support activities,Farming services,2269.674648
Business support activities,Forestry,7849.171012
Business support activities,Aquaculture,57561.31144
Business support activities,Extractive fishing,2144.761487
Business support activities,Coal mining,277.1523596
Business support activities,Crude oil and gas mining,1346.487678
Business support activities,Copper mining,462297.5175
Business support activities,Iron mining,28617.86211
Business support activities,Mining of other metals,14919.79707
Business support activities,Other mining activities and mining services,8824.588653
Business support activities,Processing and preserving of meat,36550.59741
Business support activities,Processing and preserving of fishmeal and fish oil,6608.456944
Business support activities,Processing and preserving of fish,16615.90623
Business support activities,Processing and preserving of fruits and vegetables,12179.40433
Business support activities,Manufacture of vegetable and animal oils,1563.096892
Business support activities,Manufacture of dairy products,39340.80814
Business support activities,Manufacture of grain mill products,6655.958462
Business support activities,Manufacture of prepared animal feeds,14007.81681
Business support activities,Manufacture of bakery products,15159.71565
Business support activities,Manufacture of macaroni noodles and other related products,5227.428036
Business support activities,Manufacture of other food products,17647.58575
Business support activities,Distilling rectifying and blending of spirits,5918.974124
Business support activities,Manufacture of wines,45731.51288
Business support activities,Manufacture of beers,9002.153418
Business support activities,Manufacture of soft drinks,25755.39878
Business support activities,Manufacture of tobacco products,3876.342895
Business support activities,Manufacture of textile products,2558.555216
Business support activities,Manufacture of wearing apparel,12369.26245
Business support activities,Manufacture of leather products,292.193272
Business support activities,Manufacture of footwear,974.1588482
Business support activities,Sawmilling and planing of wood,12116.64934
Business support activities,Manufacture of wood products,56113.87302
Business support activities,Manufacture of pulp and paper,86550.47993
Business support activities,Manufacture of containers of paper,9723.125883
Business support activities,Manufacture of other paper products,26653.73732
Business support activities,Printing,45279.37851
Business support activities,Manufacture of refined petroleum products,21211.33513
Business support activities,Manufacture of basic chemicals,33146.14512
Business support activities,Manufacture of paints,2444.964695
Business support activities,Manufacture of pharmaceutical products,12636.51005
Business support activities,Manufacture of soap detergents and toilet products,5909.417747
Business support activities,Manufacture of other chemical products,3863.760126
Business support activities,Manufacture of rubber products,3983.086912
Business support activities,Manufacture of plastic products,12178.68138
Business support activities,Manufacture of glass and glass products,4293.524193
Business support activities,Manufacture of cement,7147.24756
Business support activities,Manufacture of concrete and concrete products,21869.6575
Business support activities,Manufacture of basic iron and steel,4872.406999
Business support activities,Manufacture of other basic metals,2311.837273
Business support activities,Manufacture of fabricated metal products,22936.94616
Business support activities,Manufacture of industrial and domestic machinery and equipment,8546.523755
Business support activities,Manufacture of electric and electronic machinery and equipment,4402.316196
Business support activities,Manufacture of transport equipment,39980.25756
Business support activities,Manufacture of furniture,4444.238052
Business support activities,Repair and installation of machinery and other manufacturing,15635.28003
Business support activities,Electric power generation,52619.9548
Business support activities,Transmission of electric power,2692.41499
Business support activities,Distribution of electric power,45172.7807
Business support activities,Gas and steam manufacture and supply,16879.04624
Business support activities,Water collection treatment and supply,35627.89371
Business support activities,Waste collection and recycling activities,3665.519834
Business support activities,Construction of residential buildings,24345.23063
Business support activities,Construction of non-residential buildings,14039.37951
Business support activities,Civil engineering,27305.38447
Business support activities,Specialized construction activities,21603.70765
Business support activities,Wholesale and retail trade of motor vehicles,64247.10206
Business support activities,Wholesale trade,659694.9913
Business support activities,Retail trade,630680.9431
Business support activities,Accommodation,33006.28525
Business support activities,Food and beverage service activities,31496.13735
Business support activities,Transport via railways,13018.78274
Business support activities,Other passenger land transport,82968.36574
Business support activities,Freight transport by road,95782.41369
Business support activities,Transport via pipeline,1167.870045
Business support activities,Water transport,8979.896444
Business support activities,Air transport,133565.9384
Business support activities,Warehousing,28382.19837
Business support activities,Support activities for land transport,52371.31615
Business support activities,Other support activities for transport,84395.78561
Business support activities,Postal and courier activities,6127.051757
Business support activities,Wireless telecommunications,232714.1081
Business support activities,Wired telecommunications,87292.84856
Business support activities,Other telecommunications activities,97056.94922
Business support activities,Information services activities,20315.99938
Business support activities,Publishing activities,30370.70906
Business support activities,Banking monetary intermediation,255701.894
Business support activities,Insurance activities,159749.176
Business support activities,Auxiliary financial activities,209155.8757
Business support activities,Real estate activities,79381.31468
Business support activities,Housing services,0
Business support activities,Legal and accounting services,16043.95782
Business support activities,Architectural and engineering services,64081.24371
Business support activities,Other professional activities,57385.81859
Business support activities,Rental and leasing activities,19796.53207
Business support activities,Business support activities,237752.3453
Business support activities,Public administration,282419.662
Business support activities,Public education,46414.43095
Business support activities,Private education,135832.7989
Business support activities,Public health activities,128142.3995
Business support activities,Private health activities,39408.35889
Business support activities,Activities of organizations,8676.992542
Business support activities,Artistic and entertainment activities,22013.85896
Business support activities,Other personal services,6339.870481
Public administration,Cultivation of annual crops,1432.556047
Public administration,Cultivation of vegetables,147.2943711
Public administration,Cultivation of grapes,663.6158642
Public administration,Cultivation of other fruits,897.9489099
Public administration,Cattle bredding,901.5033957
Public administration,Pigs breeding,409.6061631
Public administration,Chicken breeding,796.9948365
Public administration,Breeding of other animals,129.8154427
Public administration,Farming services,27.21350297
Public administration,Forestry,2768.324761
Public administration,Aquaculture,1520.829311
Public administration,Extractive fishing,2916.048962
Public administration,Coal mining,34.18442175
Public administration,Crude oil and gas mining,40.60686306
Public administration,Copper mining,35156.83114
Public administration,Iron mining,2471.557656
Public administration,Mining of other metals,1210.363278
Public administration,Other mining activities and mining services,1947.632454
Public administration,Processing and preserving of meat,5194.92168
Public administration,Processing and preserving of fishmeal and fish oil,1582.996988
Public administration,Processing and preserving of fish,6438.421575
Public administration,Processing and preserving of fruits and vegetables,3532.791849
Public administration,Manufacture of vegetable and animal oils,1128.371632
Public administration,Manufacture of dairy products,3387.285227
Public administration,Manufacture of grain mill products,1381.440833
Public administration,Manufacture of prepared animal feeds,2600.742592
Public administration,Manufacture of bakery products,1837.488385
Public administration,Manufacture of macaroni noodles and other related products,173.513886
Public administration,Manufacture of other food products,2735.011375
Public administration,Distilling rectifying and blending of spirits,422.2025362
Public administration,Manufacture of wines,6863.60888
Public administration,Manufacture of beers,1082.329613
Public administration,Manufacture of soft drinks,2726.458226
Public administration,Manufacture of tobacco products,207.1541346
Public administration,Manufacture of textile products,1053.115154
Public administration,Manufacture of wearing apparel,1728.824437
Public administration,Manufacture of leather products,156.3521415
Public administration,Manufacture of footwear,371.8564255
Public administration,Sawmilling and planing of wood,5882.006636
Public administration,Manufacture of wood products,3613.242611
Public administration,Manufacture of pulp and paper,13934.14441
Public administration,Manufacture of containers of paper,1342.392758
Public administration,Manufacture of other paper products,12097.65353
Public administration,Printing,840.1904859
Public administration,Manufacture of refined petroleum products,1821.93832
Public administration,Manufacture of basic chemicals,6536.776166
Public administration,Manufacture of paints,375.2607252
Public administration,Manufacture of pharmaceutical products,4096.565725
Public administration,Manufacture of soap detergents and toilet products,1949.126085
Public administration,Manufacture of other chemical products,1281.854214
Public administration,Manufacture of rubber products,656.0499014
Public administration,Manufacture of plastic products,6999.795483
Public administration,Manufacture of glass and glass products,898.9639688
Public administration,Manufacture of cement,452.3739078
Public administration,Manufacture of concrete and concrete products,1950.969661
Public administration,Manufacture of basic iron and steel,1714.727662
Public administration,Manufacture of other basic metals,732.7789395
Public administration,Manufacture of fabricated metal products,3285.035381
Public administration,Manufacture of industrial and domestic machinery and equipment,1390.528821
Public administration,Manufacture of electric and electronic machinery and equipment,928.3356619
Public administration,Manufacture of transport equipment,576.0418943
Public administration,Manufacture of furniture,1323.752852
Public administration,Repair and installation of machinery and other manufacturing,784.3054251
Public administration,Electric power generation,2262.60104
Public administration,Transmission of electric power,81.64888285
Public administration,Distribution of electric power,405.3038703
Public administration,Gas and steam manufacture and supply,1523.783553
Public administration,Water collection treatment and supply,11329.36737
Public administration,Waste collection and recycling activities,3461.648975
Public administration,Construction of residential buildings,1722.312988
Public administration,Construction of non-residential buildings,654.4473452
Public administration,Civil engineering,14855.49851
Public administration,Specialized construction activities,402.3248022
Public administration,Wholesale and retail trade of motor vehicles,12157.38069
Public administration,Wholesale trade,87069.79041
Public administration,Retail trade,40109.30584
Public administration,Accommodation,1461.406149
Public administration,Food and beverage service activities,5488.890715
Public administration,Transport via railways,2990.120332
Public administration,Other passenger land transport,15033.63115
Public administration,Freight transport by road,5297.019636
Public administration,Transport via pipeline,17.05494979
Public administration,Water transport,5471.234141
Public administration,Air transport,20913.19561
Public administration,Warehousing,1098.30737
Public administration,Support activities for land transport,16612.40095
Public administration,Other support activities for transport,20191.19485
Public administration,Postal and courier activities,253.6682908
Public administration,Wireless telecommunications,3051.28783
Public administration,Wired telecommunications,1268.670447
Public administration,Other telecommunications activities,1369.205505
Public administration,Information services activities,6061.895546
Public administration,Publishing activities,5168.82614
Public administration,Banking monetary intermediation,7090.502318
Public administration,Insurance activities,2950.946543
Public administration,Auxiliary financial activities,1728.189823
Public administration,Real estate activities,4402.128588
Public administration,Housing services,24.57211158
Public administration,Legal and accounting services,2755.80855
Public administration,Architectural and engineering services,5671.929687
Public administration,Other professional activities,9259.260436
Public administration,Rental and leasing activities,1789.970973
Public administration,Business support activities,7715.99189
Public administration,Public administration,36903.86245
Public administration,Public education,3514.250702
Public administration,Private education,3693.070664
Public administration,Public health activities,1357.905051
Public administration,Private health activities,27890.83364
Public administration,Activities of organizations,2118.052297
Public administration,Artistic and entertainment activities,3413.363904
Public administration,Other personal services,830.3419574
Public education,Cultivation of annual crops,96.06658511
Public education,Cultivation of vegetables,35.81055499
Public education,Cultivation of grapes,51.96512973
Public education,Cultivation of other fruits,137.5941995
Public education,Cattle bredding,113.5455709
Public education,Pigs breeding,45.98190286
Public education,Chicken breeding,85.91260111
Public education,Breeding of other animals,11.10902661
Public education,Farming services,18.38191542
Public education,Forestry,20.33668983
Public education,Aquaculture,69.27134207
Public education,Extractive fishing,0.92004362
Public education,Coal mining,7.038979559
Public education,Crude oil and gas mining,4.674892511
Public education,Copper mining,7595.439673
Public education,Iron mining,563.5909762
Public education,Mining of other metals,464.9020133
Public education,Other mining activities and mining services,365.7169186
Public education,Processing and preserving of meat,1757.906963
Public education,Processing and preserving of fishmeal and fish oil,80.88428013
Public education,Processing and preserving of fish,1210.262606
Public education,Processing and preserving of fruits and vegetables,606.7239561
Public education,Manufacture of vegetable and animal oils,150.3527654
Public education,Manufacture of dairy products,905.5797937
Public education,Manufacture of grain mill products,561.5220575
Public education,Manufacture of prepared animal feeds,696.2909163
Public education,Manufacture of bakery products,701.6719429
Public education,Manufacture of macaroni noodles and other related products,15.79577188
Public education,Manufacture of other food products,882.4825672
Public education,Distilling rectifying and blending of spirits,298.9135497
Public education,Manufacture of wines,1743.789143
Public education,Manufacture of beers,804.3914464
Public education,Manufacture of soft drinks,1524.082206
Public education,Manufacture of tobacco products,238.0020942
Public education,Manufacture of textile products,278.7477493
Public education,Manufacture of wearing apparel,785.9129491
Public education,Manufacture of leather products,19.96677375
Public education,Manufacture of footwear,69.0478036
Public education,Sawmilling and planing of wood,1088.140955
Public education,Manufacture of wood products,989.7103196
Public education,Manufacture of pulp and paper,983.7868921
Public education,Manufacture of containers of paper,357.3497568
Public education,Manufacture of other paper products,738.9966522
Public education,Printing,575.3185265
Public education,Manufacture of refined petroleum products,713.5104109
Public education,Manufacture of basic chemicals,868.8379936
Public education,Manufacture of paints,273.5648965
Public education,Manufacture of pharmaceutical products,2283.631454
Public education,Manufacture of soap detergents and toilet products,893.861446
Public education,Manufacture of other chemical products,514.3177044
Public education,Manufacture of rubber products,14.23220333
Public education,Manufacture of plastic products,507.720935
Public education,Manufacture of glass and glass products,123.1936479
Public education,Manufacture of cement,478.2034561
Public education,Manufacture of concrete and concrete products,1259.740983
Public education,Manufacture of basic iron and steel,296.6926208
Public education,Manufacture of other basic metals,141.3471499
Public education,Manufacture of fabricated metal products,1632.710816
Public education,Manufacture of industrial and domestic machinery and equipment,786.4875628
Public education,Manufacture of electric and electronic machinery and equipment,259.0985084
Public education,Manufacture of transport equipment,468.418727
Public education,Manufacture of furniture,494.8687374
Public education,Repair and installation of machinery and other manufacturing,333.5662954
Public education,Electric power generation,647.6525321
Public education,Transmission of electric power,13.22905127
Public education,Distribution of electric power,89.77517754
Public education,Gas and steam manufacture and supply,1720.354247
Public education,Water collection treatment and supply,166.7088057
Public education,Waste collection and recycling activities,78.56833258
Public education,Construction of residential buildings,1861.189301
Public education,Construction of non-residential buildings,680.1369179
Public education,Civil engineering,1282.832294
Public education,Specialized construction activities,592.9622668
Public education,Wholesale and retail trade of motor vehicles,1523.229941
Public education,Wholesale trade,10053.44581
Public education,Retail trade,6718.747956
Public education,Accommodation,219.6290054
Public education,Food and beverage service activities,699.580407
Public education,Transport via railways,102.1051295
Public education,Other passenger land transport,320.3453669
Public education,Freight transport by road,482.5168263
Public education,Transport via pipeline,3.527301699
Public education,Water transport,62.08004662
Public education,Air transport,696.419641
Public education,Warehousing,277.7245894
Public education,Support activities for land transport,447.7984938
Public education,Other support activities for transport,255.1292075
Public education,Postal and courier activities,42.66014182
Public education,Wireless telecommunications,1317.918845
Public education,Wired telecommunications,454.2617883
Public education,Other telecommunications activities,1536.374903
Public education,Information services activities,2651.628147
Public education,Publishing activities,1109.686181
Public education,Banking monetary intermediation,4402.779159
Public education,Insurance activities,2564.422447
Public education,Auxiliary financial activities,1137.752162
Public education,Real estate activities,2463.215117
Public education,Housing services,0
Public education,Legal and accounting services,1632.091638
Public education,Architectural and engineering services,5865.096759
Public education,Other professional activities,7162.128445
Public education,Rental and leasing activities,1119.89947
Public education,Business support activities,5206.958997
Public education,Public administration,2037.717595
Public education,Public education,1216.232798
Public education,Private education,2275.782118
Public education,Public health activities,873.8366473
Public education,Private health activities,2268.533202
Public education,Activities of organizations,88.50704829
Public education,Artistic and entertainment activities,666.5184945
Public education,Other personal services,92.25290197
Private education,Cultivation of annual crops,222.4106953
Private education,Cultivation of vegetables,83.00213851
Private education,Cultivation of grapes,121.8756068
Private education,Cultivation of other fruits,326.8830031
Private education,Cattle bredding,255.909497
Private education,Pigs breeding,103.1217709
Private education,Chicken breeding,189.8673002
Private education,Breeding of other animals,24.88482716
Private education,Farming services,17.51290242
Private education,Forestry,47.57278423
Private education,Aquaculture,157.4719411
Private education,Extractive fishing,4.223806083
Private education,Coal mining,4.072933655
Private education,Crude oil and gas mining,10.96046873
Private education,Copper mining,7042.661803
Private education,Iron mining,105.0375012
Private education,Mining of other metals,65.94298574
Private education,Other mining activities and mining services,91.39800836
Private education,Processing and preserving of meat,3912.238128
Private education,Processing and preserving of fishmeal and fish oil,177.0858216
Private education,Processing and preserving of fish,2654.285576
Private education,Processing and preserving of fruits and vegetables,1374.672219
Private education,Manufacture of vegetable and animal oils,329.7400333
Private education,Manufacture of dairy products,2056.943134
Private education,Manufacture of grain mill products,1213.022919
Private education,Manufacture of prepared animal feeds,1566.094248
Private education,Manufacture of bakery products,1589.565491
Private education,Manufacture of macaroni noodles and other related products,38.8339342
Private education,Manufacture of other food products,1955.53122
Private education,Distilling rectifying and blending of spirits,559.3391661
Private education,Manufacture of wines,3827.706644
Private education,Manufacture of beers,1782.661173
Private education,Manufacture of soft drinks,3451.337618
Private education,Manufacture of tobacco products,476.6252644
Private education,Manufacture of textile products,625.8791256
Private education,Manufacture of wearing apparel,1749.664465
Private education,Manufacture of leather products,46.24982323
Private education,Manufacture of footwear,155.1673288
Private education,Sawmilling and planing of wood,2420.257494
Private education,Manufacture of wood products,2209.165172
Private education,Manufacture of pulp and paper,2151.434487
Private education,Manufacture of containers of paper,812.0123896
Private education,Manufacture of other paper products,1639.614829
Private education,Printing,1209.296655
Private education,Manufacture of refined petroleum products,1601.671049
Private education,Manufacture of basic chemicals,1750.59153
Private education,Manufacture of paints,617.1661356
Private education,Manufacture of pharmaceutical products,5061.388807
Private education,Manufacture of soap detergents and toilet products,2030.356843
Private education,Manufacture of other chemical products,1134.79272
Private education,Manufacture of rubber products,29.90018693
Private education,Manufacture of plastic products,1152.397214
Private education,Manufacture of glass and glass products,282.2150709
Private education,Manufacture of cement,1013.183144
Private education,Manufacture of concrete and concrete products,2827.752013
Private education,Manufacture of basic iron and steel,667.9970985
Private education,Manufacture of other basic metals,308.7463094
Private education,Manufacture of fabricated metal products,3567.762561
Private education,Manufacture of industrial and domestic machinery and equipment,1641.520037
Private education,Manufacture of electric and electronic machinery and equipment,584.4382267
Private education,Manufacture of transport equipment,1023.736276
Private education,Manufacture of furniture,1038.502745
Private education,Repair and installation of machinery and other manufacturing,785.1364855
Private education,Electric power generation,1336.630307
Private education,Transmission of electric power,23.74527097
Private education,Distribution of electric power,199.754547
Private education,Gas and steam manufacture and supply,3871.56616
Private education,Water collection treatment and supply,244.803863
Private education,Waste collection and recycling activities,166.6443107
Private education,Construction of residential buildings,3934.814275
Private education,Construction of non-residential buildings,1395.261009
Private education,Civil engineering,373.8272768
Private education,Specialized construction activities,207.6015142
Private education,Wholesale and retail trade of motor vehicles,3465.260357
Private education,Wholesale trade,21393.84526
Private education,Retail trade,16307.97918
Private education,Accommodation,608.9687891
Private education,Food and beverage service activities,1945.992321
Private education,Transport via railways,227.6609732
Private education,Other passenger land transport,567.4711016
Private education,Freight transport by road,850.9221868
Private education,Transport via pipeline,4.426350939
Private education,Water transport,130.0412946
Private education,Air transport,1637.974399
Private education,Warehousing,676.5238885
Private education,Support activities for land transport,1010.483145
Private education,Other support activities for transport,530.3189807
Private education,Postal and courier activities,82.33589491
Private education,Wireless telecommunications,3133.575871
Private education,Wired telecommunications,1065.544414
Private education,Other telecommunications activities,3396.524717
Private education,Information services activities,6214.576699
Private education,Publishing activities,2530.047047
Private education,Banking monetary intermediation,10119.14906
Private education,Insurance activities,5679.453697
Private education,Auxiliary financial activities,2546.404729
Private education,Real estate activities,5604.516958
Private education,Housing services,0
Private education,Legal and accounting services,3754.619
Private education,Architectural and engineering services,7801.296603
Private education,Other professional activities,15901.65937
Private education,Rental and leasing activities,2483.713688
Private education,Business support activities,11753.9884
Private education,Public administration,4568.332074
Private education,Public education,3001.874876
Private education,Private education,5264.595277
Private education,Public health activities,1975.939562
Private education,Private health activities,4321.46198
Private education,Activities of organizations,217.3097046
Private education,Artistic and entertainment activities,1583.512179
Private education,Other personal services,212.0204064
Public health activities,Cultivation of annual crops,2.254460132
Public health activities,Cultivation of vegetables,1.191933887
Public health activities,Cultivation of grapes,1.71355035
Public health activities,Cultivation of other fruits,4.843430516
Public health activities,Cattle bredding,1.929955921
Public health activities,Pigs breeding,0.802527185
Public health activities,Chicken breeding,1.384144791
Public health activities,Breeding of other animals,0.156129298
Public health activities,Farming services,0.197514291
Public health activities,Forestry,0.383005968
Public health activities,Aquaculture,4.5861778
Public health activities,Extractive fishing,0.065754279
Public health activities,Coal mining,0.005644246
Public health activities,Crude oil and gas mining,0.292128875
Public health activities,Copper mining,72.81989946
Public health activities,Iron mining,5.679283603
Public health activities,Mining of other metals,0.446045327
Public health activities,Other mining activities and mining services,0.191211685
Public health activities,Processing and preserving of meat,6.505968354
Public health activities,Processing and preserving of fishmeal and fish oil,0.216157854
Public health activities,Processing and preserving of fish,2.50265853
Public health activities,Processing and preserving of fruits and vegetables,2.374108282
Public health activities,Manufacture of vegetable and animal oils,0.468179288
Public health activities,Manufacture of dairy products,15.51203414
Public health activities,Manufacture of grain mill products,5.869003743
Public health activities,Manufacture of prepared animal feeds,4.554289439
Public health activities,Manufacture of bakery products,15.99298486
Public health activities,Manufacture of macaroni noodles and other related products,1.173527292
Public health activities,Manufacture of other food products,13.08180491
Public health activities,Distilling rectifying and blending of spirits,0.804621743
Public health activities,Manufacture of wines,15.44537016
Public health activities,Manufacture of beers,3.536773676
Public health activities,Manufacture of soft drinks,18.41279063
Public health activities,Manufacture of tobacco products,0.288231408
Public health activities,Manufacture of textile products,5.989337838
Public health activities,Manufacture of wearing apparel,17.3488462
Public health activities,Manufacture of leather products,0.337672905
Public health activities,Manufacture of footwear,1.873627396
Public health activities,Sawmilling and planing of wood,10.97738807
Public health activities,Manufacture of wood products,5.949307056
Public health activities,Manufacture of pulp and paper,7.475964157
Public health activities,Manufacture of containers of paper,2.487912788
Public health activities,Manufacture of other paper products,10.87895356
Public health activities,Printing,7.94210039
Public health activities,Manufacture of refined petroleum products,9.535463785
Public health activities,Manufacture of basic chemicals,9.560838981
Public health activities,Manufacture of paints,3.533175974
Public health activities,Manufacture of pharmaceutical products,18.90699749
Public health activities,Manufacture of soap detergents and toilet products,12.06347553
Public health activities,Manufacture of other chemical products,3.661969204
Public health activities,Manufacture of rubber products,0.71526993
Public health activities,Manufacture of plastic products,6.140388834
Public health activities,Manufacture of glass and glass products,2.880364785
Public health activities,Manufacture of cement,3.469681565
Public health activities,Manufacture of concrete and concrete products,11.50862865
Public health activities,Manufacture of basic iron and steel,1.339779243
Public health activities,Manufacture of other basic metals,2.018807776
Public health activities,Manufacture of fabricated metal products,26.80008255
Public health activities,Manufacture of industrial and domestic machinery and equipment,5.33847553
Public health activities,Manufacture of electric and electronic machinery and equipment,7.203421166
Public health activities,Manufacture of transport equipment,4.442619913
Public health activities,Manufacture of furniture,17.41186373
Public health activities,Repair and installation of machinery and other manufacturing,11.25449611
Public health activities,Electric power generation,5.300638671
Public health activities,Transmission of electric power,1.114799265
Public health activities,Distribution of electric power,8.88732451
Public health activities,Gas and steam manufacture and supply,3.681400168
Public health activities,Water collection treatment and supply,3.182040923
Public health activities,Waste collection and recycling activities,4.315002544
Public health activities,Construction of residential buildings,15.13965795
Public health activities,Construction of non-residential buildings,6.480159257
Public health activities,Civil engineering,11.9689561
Public health activities,Specialized construction activities,2.426322701
Public health activities,Wholesale and retail trade of motor vehicles,42.99647366
Public health activities,Wholesale trade,191.6144141
Public health activities,Retail trade,434.7615587
Public health activities,Accommodation,39.14499609
Public health activities,Food and beverage service activities,70.96062615
Public health activities,Transport via railways,1.186351954
Public health activities,Other passenger land transport,8.410205333
Public health activities,Freight transport by road,23.09898693
Public health activities,Transport via pipeline,0.416030326
Public health activities,Water transport,1.156128329
Public health activities,Air transport,43.12384941
Public health activities,Warehousing,14.88214431
Public health activities,Support activities for land transport,9.299945207
Public health activities,Other support activities for transport,26.94335336
Public health activities,Postal and courier activities,1.166305585
Public health activities,Wireless telecommunications,63.53432781
Public health activities,Wired telecommunications,34.65300342
Public health activities,Other telecommunications activities,9.072942725
Public health activities,Information services activities,153.8106966
Public health activities,Publishing activities,24.09554784
Public health activities,Banking monetary intermediation,145.0106802
Public health activities,Insurance activities,35.0558609
Public health activities,Auxiliary financial activities,26.90563054
Public health activities,Real estate activities,93.79373288
Public health activities,Housing services,0
Public health activities,Legal and accounting services,31.75316524
Public health activities,Architectural and engineering services,83.18697312
Public health activities,Other professional activities,129.8102915
Public health activities,Rental and leasing activities,27.42298176
Public health activities,Business support activities,100.2014028
Public health activities,Public administration,88.9840479
Public health activities,Public education,72.65909239
Public health activities,Private education,49.53635485
Public health activities,Public health activities,24.34761689
Public health activities,Private health activities,201.393286
Public health activities,Activities of organizations,15.78320532
Public health activities,Artistic and entertainment activities,36.51123252
Public health activities,Other personal services,13.67838363
Private health activities,Cultivation of annual crops,262.3620119
Private health activities,Cultivation of vegetables,304.8168441
Private health activities,Cultivation of grapes,157.9546491
Private health activities,Cultivation of other fruits,359.4213417
Private health activities,Cattle bredding,184.3774077
Private health activities,Pigs breeding,87.8550351
Private health activities,Chicken breeding,164.8069157
Private health activities,Breeding of other animals,15.35023322
Private health activities,Farming services,63.82627286
Private health activities,Forestry,150.6083594
Private health activities,Aquaculture,131.1654862
Private health activities,Extractive fishing,72.31131217
Private health activities,Coal mining,7.267982957
Private health activities,Crude oil and gas mining,18.38578776
Private health activities,Copper mining,3243.839749
Private health activities,Iron mining,261.126753
Private health activities,Mining of other metals,132.2807357
Private health activities,Other mining activities and mining services,192.7223348
Private health activities,Processing and preserving of meat,704.2918694
Private health activities,Processing and preserving of fishmeal and fish oil,11.81085633
Private health activities,Processing and preserving of fish,615.7234385
Private health activities,Processing and preserving of fruits and vegetables,315.8681253
Private health activities,Manufacture of vegetable and animal oils,71.95655906
Private health activities,Manufacture of dairy products,685.3287869
Private health activities,Manufacture of grain mill products,299.7622498
Private health activities,Manufacture of prepared animal feeds,453.3152618
Private health activities,Manufacture of bakery products,685.2551432
Private health activities,Manufacture of macaroni noodles and other related products,57.19347889
Private health activities,Manufacture of other food products,582.0755754
Private health activities,Distilling rectifying and blending of spirits,60.32325399
Private health activities,Manufacture of wines,825.4244001
Private health activities,Manufacture of beers,217.7502121
Private health activities,Manufacture of soft drinks,779.9590781
Private health activities,Manufacture of tobacco products,129.468245
Private health activities,Manufacture of textile products,247.9113129
Private health activities,Manufacture of wearing apparel,599.5755944
Private health activities,Manufacture of leather products,19.41608517
Private health activities,Manufacture of footwear,99.80685871
Private health activities,Sawmilling and planing of wood,513.4679759
Private health activities,Manufacture of wood products,400.2923103
Private health activities,Manufacture of pulp and paper,700.2107848
Private health activities,Manufacture of containers of paper,191.4486937
Private health activities,Manufacture of other paper products,401.7347301
Private health activities,Printing,314.1674668
Private health activities,Manufacture of refined petroleum products,276.5648133
Private health activities,Manufacture of basic chemicals,593.793863
Private health activities,Manufacture of paints,143.6486133
Private health activities,Manufacture of pharmaceutical products,815.7862351
Private health activities,Manufacture of soap detergents and toilet products,490.9405874
Private health activities,Manufacture of other chemical products,202.1630181
Private health activities,Manufacture of rubber products,84.61139788
Private health activities,Manufacture of plastic products,444.8647808
Private health activities,Manufacture of glass and glass products,134.569948
Private health activities,Manufacture of cement,208.2961268
Private health activities,Manufacture of concrete and concrete products,597.439575
Private health activities,Manufacture of basic iron and steel,303.2553555
Private health activities,Manufacture of other basic metals,164.5158336
Private health activities,Manufacture of fabricated metal products,1073.902786
Private health activities,Manufacture of industrial and domestic machinery and equipment,341.8886495
Private health activities,Manufacture of electric and electronic machinery and equipment,210.8186562
Private health activities,Manufacture of transport equipment,222.9737969
Private health activities,Manufacture of furniture,555.0693542
Private health activities,Repair and installation of machinery and other manufacturing,300.8365356
Private health activities,Electric power generation,164.7751444
Private health activities,Transmission of electric power,25.38130334
Private health activities,Distribution of electric power,206.8251123
Private health activities,Gas and steam manufacture and supply,216.4548188
Private health activities,Water collection treatment and supply,79.13894746
Private health activities,Waste collection and recycling activities,119.7714137
Private health activities,Construction of residential buildings,1202.843447
Private health activities,Construction of non-residential buildings,483.237187
Private health activities,Civil engineering,1218.052044
Private health activities,Specialized construction activities,458.3335685
Private health activities,Wholesale and retail trade of motor vehicles,1464.368085
Private health activities,Wholesale trade,6916.623763
Private health activities,Retail trade,12099.3062
Private health activities,Accommodation,962.2672362
Private health activities,Food and beverage service activities,2325.195424
Private health activities,Transport via railways,34.51361216
Private health activities,Other passenger land transport,977.6203519
Private health activities,Freight transport by road,781.5690049
Private health activities,Transport via pipeline,9.227264206
Private health activities,Water transport,44.89929933
Private health activities,Air transport,1013.584078
Private health activities,Warehousing,344.7730432
Private health activities,Support activities for land transport,253.1811606
Private health activities,Other support activities for transport,621.7602048
Private health activities,Postal and courier activities,28.3446623
Private health activities,Wireless telecommunications,1492.475416
Private health activities,Wired telecommunications,817.599073
Private health activities,Other telecommunications activities,326.2077591
Private health activities,Information services activities,3706.833693
Private health activities,Publishing activities,671.031133
Private health activities,Banking monetary intermediation,3786.42368
Private health activities,Insurance activities,6165.953221
Private health activities,Auxiliary financial activities,1064.420231
Private health activities,Real estate activities,2221.624466
Private health activities,Housing services,0
Private health activities,Legal and accounting services,824.3853154
Private health activities,Architectural and engineering services,2092.640977
Private health activities,Other professional activities,3446.333006
Private health activities,Rental and leasing activities,697.5428591
Private health activities,Business support activities,2628.313624
Private health activities,Public administration,2417.800989
Private health activities,Public education,3003.287568
Private health activities,Private education,2073.437752
Private health activities,Public health activities,638.9203168
Private health activities,Private health activities,780086.0991
Private health activities,Activities of organizations,14171.10959
Private health activities,Artistic and entertainment activities,854.4277817
Private health activities,Other personal services,330.7962757
Activities of organizations,Cultivation of annual crops,747.6330291
Activities of organizations,Cultivation of vegetables,1008.354091
Activities of organizations,Cultivation of grapes,424.9599268
Activities of organizations,Cultivation of other fruits,880.0560302
Activities of organizations,Cattle bredding,489.6050675
Activities of organizations,Pigs breeding,244.4681624
Activities of organizations,Chicken breeding,472.6893924
Activities of organizations,Breeding of other animals,39.59153153
Activities of organizations,Farming services,209.6829143
Activities of organizations,Forestry,486.6757339
Activities of organizations,Aquaculture,87.88590745
Activities of organizations,Extractive fishing,239.6362206
Activities of organizations,Coal mining,22.87899316
Activities of organizations,Crude oil and gas mining,0.115478654
Activities of organizations,Copper mining,4792.403901
Activities of organizations,Iron mining,491.408526
Activities of organizations,Mining of other metals,318.1421691
Activities of organizations,Other mining activities and mining services,652.6202286
Activities of organizations,Processing and preserving of meat,1561.659586
Activities of organizations,Processing and preserving of fishmeal and fish oil,2.76659577
Activities of organizations,Processing and preserving of fish,1755.095308
Activities of organizations,Processing and preserving of fruits and vegetables,775.1903501
Activities of organizations,Manufacture of vegetable and animal oils,179.4808326
Activities of organizations,Manufacture of dairy products,966.9276024
Activities of organizations,Manufacture of grain mill products,468.1816387
Activities of organizations,Manufacture of prepared animal feeds,1045.860195
Activities of organizations,Manufacture of bakery products,950.2647658
Activities of organizations,Manufacture of macaroni noodles and other related products,109.457694
Activities of organizations,Manufacture of other food products,792.27098
Activities of organizations,Distilling rectifying and blending of spirits,92.41982058
Activities of organizations,Manufacture of wines,1322.393562
Activities of organizations,Manufacture of beers,289.6119058
Activities of organizations,Manufacture of soft drinks,933.2903837
Activities of organizations,Manufacture of tobacco products,398.0254603
Activities of organizations,Manufacture of textile products,306.3322245
Activities of organizations,Manufacture of wearing apparel,444.5267824
Activities of organizations,Manufacture of leather products,37.3846057
Activities of organizations,Manufacture of footwear,188.736519
Activities of organizations,Sawmilling and planing of wood,706.8622485
Activities of organizations,Manufacture of wood products,731.5244778
Activities of organizations,Manufacture of pulp and paper,1702.621874
Activities of organizations,Manufacture of containers of paper,361.0604363
Activities of organizations,Manufacture of other paper products,385.2644366
Activities of organizations,Printing,317.4960091
Activities of organizations,Manufacture of refined petroleum products,23.65487094
Activities of organizations,Manufacture of basic chemicals,1154.217615
Activities of organizations,Manufacture of paints,163.6445428
Activities of organizations,Manufacture of pharmaceutical products,861.5159203
Activities of organizations,Manufacture of soap detergents and toilet products,552.7600586
Activities of organizations,Manufacture of other chemical products,319.9415166
Activities of organizations,Manufacture of rubber products,245.1969086
Activities of organizations,Manufacture of plastic products,995.7365333
Activities of organizations,Manufacture of glass and glass products,221.6753416
Activities of organizations,Manufacture of cement,366.273925
Activities of organizations,Manufacture of concrete and concrete products,925.7584652
Activities of organizations,Manufacture of basic iron and steel,928.0063205
Activities of organizations,Manufacture of other basic metals,401.703846
Activities of organizations,Manufacture of fabricated metal products,1311.851005
Activities of organizations,Manufacture of industrial and domestic machinery and equipment,633.484058
Activities of organizations,Manufacture of electric and electronic machinery and equipment,114.403189
Activities of organizations,Manufacture of transport equipment,333.0563997
Activities of organizations,Manufacture of furniture,432.5073327
Activities of organizations,Repair and installation of machinery and other manufacturing,106.6973232
Activities of organizations,Electric power generation,20.37254115
Activities of organizations,Transmission of electric power,0.201757244
Activities of organizations,Distribution of electric power,1.533270897
Activities of organizations,Gas and steam manufacture and supply,61.18039171
Activities of organizations,Water collection treatment and supply,3.244745355
Activities of organizations,Waste collection and recycling activities,65.86474684
Activities of organizations,Construction of residential buildings,2653.825928
Activities of organizations,Construction of non-residential buildings,1017.147792
Activities of organizations,Civil engineering,3132.900978
Activities of organizations,Specialized construction activities,1136.181433
Activities of organizations,Wholesale and retail trade of motor vehicles,1522.463898
Activities of organizations,Wholesale trade,7410.874909
Activities of organizations,Retail trade,7864.262334
Activities of organizations,Accommodation,281.9465517
Activities of organizations,Food and beverage service activities,1805.662606
Activities of organizations,Transport via railways,3.408148813
Activities of organizations,Other passenger land transport,2367.104028
Activities of organizations,Freight transport by road,9.602013251
Activities of organizations,Transport via pipeline,0.006318008
Activities of organizations,Water transport,42.48829226
Activities of organizations,Air transport,19.09416873
Activities of organizations,Warehousing,8.827613555
Activities of organizations,Support activities for land transport,14.6541364
Activities of organizations,Other support activities for transport,18.24347192
Activities of organizations,Postal and courier activities,1.140897968
Activities of organizations,Wireless telecommunications,40.8611337
Activities of organizations,Wired telecommunications,10.59052029
Activities of organizations,Other telecommunications activities,52.37874611
Activities of organizations,Information services activities,70.25108952
Activities of organizations,Publishing activities,234.7151997
Activities of organizations,Banking monetary intermediation,749.9643599
Activities of organizations,Insurance activities,1588.563845
Activities of organizations,Auxiliary financial activities,1382.390814
Activities of organizations,Real estate activities,76.8379771
Activities of organizations,Housing services,0
Activities of organizations,Legal and accounting services,55.41460807
Activities of organizations,Architectural and engineering services,108.0792232
Activities of organizations,Other professional activities,231.4361179
Activities of organizations,Rental and leasing activities,35.31442561
Activities of organizations,Business support activities,171.137964
Activities of organizations,Public administration,56.42093467
Activities of organizations,Public education,38.19786751
Activities of organizations,Private education,77.01723439
Activities of organizations,Public health activities,26.67573891
Activities of organizations,Private health activities,48.78318437
Activities of organizations,Activities of organizations,1.322456612
Activities of organizations,Artistic and entertainment activities,20.00727055
Activities of organizations,Other personal services,86.59930129
Artistic and entertainment activities,Cultivation of annual crops,9.812763561
Artistic and entertainment activities,Cultivation of vegetables,4.831715633
Artistic and entertainment activities,Cultivation of grapes,6.169286976
Artistic and entertainment activities,Cultivation of other fruits,17.33581234
Artistic and entertainment activities,Cattle bredding,8.85206454
Artistic and entertainment activities,Pigs breeding,3.662830904
Artistic and entertainment activities,Chicken breeding,5.897663161
Artistic and entertainment activities,Breeding of other animals,1.085591271
Artistic and entertainment activities,Farming services,5.273032619
Artistic and entertainment activities,Forestry,3.983504318
Artistic and entertainment activities,Aquaculture,4.144415026
Artistic and entertainment activities,Extractive fishing,4.651568924
Artistic and entertainment activities,Coal mining,0.327874726
Artistic and entertainment activities,Crude oil and gas mining,0.84461946
Artistic and entertainment activities,Copper mining,353.9043031
Artistic and entertainment activities,Iron mining,1.094871497
Artistic and entertainment activities,Mining of other metals,2.857147951
Artistic and entertainment activities,Other mining activities and mining services,3.54998093
Artistic and entertainment activities,Processing and preserving of meat,130.0228378
Artistic and entertainment activities,Processing and preserving of fishmeal and fish oil,5.979586132
Artistic and entertainment activities,Processing and preserving of fish,79.43266035
Artistic and entertainment activities,Processing and preserving of fruits and vegetables,48.51539506
Artistic and entertainment activities,Manufacture of vegetable and animal oils,11.84410582
Artistic and entertainment activities,Manufacture of dairy products,94.59149494
Artistic and entertainment activities,Manufacture of grain mill products,47.68914941
Artistic and entertainment activities,Manufacture of prepared animal feeds,65.9550024
Artistic and entertainment activities,Manufacture of bakery products,81.03166419
Artistic and entertainment activities,Manufacture of macaroni noodles and other related products,1.775369315
Artistic and entertainment activities,Manufacture of other food products,88.72725641
Artistic and entertainment activities,Distilling rectifying and blending of spirits,16.53142314
Artistic and entertainment activities,Manufacture of wines,135.7413584
Artistic and entertainment activities,Manufacture of beers,58.81379934
Artistic and entertainment activities,Manufacture of soft drinks,135.4210906
Artistic and entertainment activities,Manufacture of tobacco products,14.0044586
Artistic and entertainment activities,Manufacture of textile products,34.4114103
Artistic and entertainment activities,Manufacture of wearing apparel,101.2773912
Artistic and entertainment activities,Manufacture of leather products,1.706402056
Artistic and entertainment activities,Manufacture of footwear,6.822229517
Artistic and entertainment activities,Sawmilling and planing of wood,87.45944161
Artistic and entertainment activities,Manufacture of wood products,71.74222762
Artistic and entertainment activities,Manufacture of pulp and paper,77.88118576
Artistic and entertainment activities,Manufacture of containers of paper,36.32753715
Artistic and entertainment activities,Manufacture of other paper products,65.52320463
Artistic and entertainment activities,Printing,54.86306309
Artistic and entertainment activities,Manufacture of refined petroleum products,68.86997059
Artistic and entertainment activities,Manufacture of basic chemicals,75.10324186
Artistic and entertainment activities,Manufacture of paints,23.55975697
Artistic and entertainment activities,Manufacture of pharmaceutical products,177.320879
Artistic and entertainment activities,Manufacture of soap detergents and toilet products,86.1182324
Artistic and entertainment activities,Manufacture of other chemical products,34.00756167
Artistic and entertainment activities,Manufacture of rubber products,2.584757855
Artistic and entertainment activities,Manufacture of plastic products,38.21417463
Artistic and entertainment activities,Manufacture of glass and glass products,13.38010628
Artistic and entertainment activities,Manufacture of cement,33.36982348
Artistic and entertainment activities,Manufacture of concrete and concrete products,96.18185397
Artistic and entertainment activities,Manufacture of basic iron and steel,20.71013094
Artistic and entertainment activities,Manufacture of other basic metals,13.15770943
Artistic and entertainment activities,Manufacture of fabricated metal products,150.4054534
Artistic and entertainment activities,Manufacture of industrial and domestic machinery and equipment,50.78793874
Artistic and entertainment activities,Manufacture of electric and electronic machinery and equipment,25.63123164
Artistic and entertainment activities,Manufacture of transport equipment,36.20514204
Artistic and entertainment activities,Manufacture of furniture,68.838881
Artistic and entertainment activities,Repair and installation of machinery and other manufacturing,25.14047635
Artistic and entertainment activities,Electric power generation,42.13743719
Artistic and entertainment activities,Transmission of electric power,1.32136156
Artistic and entertainment activities,Distribution of electric power,19.12905671
Artistic and entertainment activities,Gas and steam manufacture and supply,112.7790153
Artistic and entertainment activities,Water collection treatment and supply,9.844805049
Artistic and entertainment activities,Waste collection and recycling activities,9.803375656
Artistic and entertainment activities,Construction of residential buildings,134.1980176
Artistic and entertainment activities,Construction of non-residential buildings,56.03665666
Artistic and entertainment activities,Civil engineering,84.64162682
Artistic and entertainment activities,Specialized construction activities,79.66975527
Artistic and entertainment activities,Wholesale and retail trade of motor vehicles,114.3604647
Artistic and entertainment activities,Wholesale trade,752.0198612
Artistic and entertainment activities,Retail trade,678.0631256
Artistic and entertainment activities,Accommodation,301.3924024
Artistic and entertainment activities,Food and beverage service activities,406.7090537
Artistic and entertainment activities,Transport via railways,8.043230495
Artistic and entertainment activities,Other passenger land transport,37.22429357
Artistic and entertainment activities,Freight transport by road,77.29836585
Artistic and entertainment activities,Transport via pipeline,0.250411966
Artistic and entertainment activities,Water transport,9.609520423
Artistic and entertainment activities,Air transport,119.6668758
Artistic and entertainment activities,Warehousing,16.54343725
Artistic and entertainment activities,Support activities for land transport,32.88552138
Artistic and entertainment activities,Other support activities for transport,39.45216349
Artistic and entertainment activities,Postal and courier activities,2.086136027
Artistic and entertainment activities,Wireless telecommunications,99.54858249
Artistic and entertainment activities,Wired telecommunications,88.81002393
Artistic and entertainment activities,Other telecommunications activities,110.2834399
Artistic and entertainment activities,Information services activities,484.0451785
Artistic and entertainment activities,Publishing activities,87998.29328
Artistic and entertainment activities,Banking monetary intermediation,540.4102799
Artistic and entertainment activities,Insurance activities,323.3442404
Artistic and entertainment activities,Auxiliary financial activities,197.3734031
Artistic and entertainment activities,Real estate activities,195.1920226
Artistic and entertainment activities,Housing services,0
Artistic and entertainment activities,Legal and accounting services,113.5741155
Artistic and entertainment activities,Architectural and engineering services,263.5326958
Artistic and entertainment activities,Other professional activities,656.6808197
Artistic and entertainment activities,Rental and leasing activities,89.95546749
Artistic and entertainment activities,Business support activities,709.278612
Artistic and entertainment activities,Public administration,25277.82331
Artistic and entertainment activities,Public education,124.7890403
Artistic and entertainment activities,Private education,260.0485574
Artistic and entertainment activities,Public health activities,266.0832018
Artistic and entertainment activities,Private health activities,188.2489956
Artistic and entertainment activities,Activities of organizations,212.712118
Artistic and entertainment activities,Artistic and entertainment activities,40347.23134
Artistic and entertainment activities,Other personal services,21.59722466
Other personal services,Cultivation of annual crops,5.841620729
Other personal services,Cultivation of vegetables,4.231334487
Other personal services,Cultivation of grapes,4.107585288
Other personal services,Cultivation of other fruits,10.68889661
Other personal services,Cattle bredding,3.705438368
Other personal services,Pigs breeding,1.380400463
Other personal services,Chicken breeding,1.524436151
Other personal services,Breeding of other animals,0.65631022
Other personal services,Farming services,3.20119412
Other personal services,Forestry,66.96141421
Other personal services,Aquaculture,13.40459764
Other personal services,Extractive fishing,99.82351331
Other personal services,Coal mining,7.345750094
Other personal services,Crude oil and gas mining,41.76537563
Other personal services,Copper mining,6265.494454
Other personal services,Iron mining,582.7074758
Other personal services,Mining of other metals,540.0693915
Other personal services,Other mining activities and mining services,399.0387275
Other personal services,Processing and preserving of meat,56.08242742
Other personal services,Processing and preserving of fishmeal and fish oil,30.97925595
Other personal services,Processing and preserving of fish,150.7987851
Other personal services,Processing and preserving of fruits and vegetables,74.14029469
Other personal services,Manufacture of vegetable and animal oils,3.905332425
Other personal services,Manufacture of dairy products,122.5224622
Other personal services,Manufacture of grain mill products,112.4079375
Other personal services,Manufacture of prepared animal feeds,126.3206809
Other personal services,Manufacture of bakery products,196.7048398
Other personal services,Manufacture of macaroni noodles and other related products,2.387754909
Other personal services,Manufacture of other food products,233.5616316
Other personal services,Distilling rectifying and blending of spirits,1.42495557
Other personal services,Manufacture of wines,205.9013548
Other personal services,Manufacture of beers,49.61136523
Other personal services,Manufacture of soft drinks,224.8062177
Other personal services,Manufacture of tobacco products,0.633231023
Other personal services,Manufacture of textile products,64.29506146
Other personal services,Manufacture of wearing apparel,286.0562253
Other personal services,Manufacture of leather products,9.57393915
Other personal services,Manufacture of footwear,22.81083311
Other personal services,Sawmilling and planing of wood,71.02769191
Other personal services,Manufacture of wood products,358.1459787
Other personal services,Manufacture of pulp and paper,535.9565521
Other personal services,Manufacture of containers of paper,260.0811481
Other personal services,Manufacture of other paper products,257.571874
Other personal services,Printing,171.0541295
Other personal services,Manufacture of refined petroleum products,80.0529212
Other personal services,Manufacture of basic chemicals,224.4967568
Other personal services,Manufacture of paints,21.09769347
Other personal services,Manufacture of pharmaceutical products,226.7038637
Other personal services,Manufacture of soap detergents and toilet products,208.2727656
Other personal services,Manufacture of other chemical products,95.06938416
Other personal services,Manufacture of rubber products,65.4803266
Other personal services,Manufacture of plastic products,198.3521613
Other personal services,Manufacture of glass and glass products,155.2411986
Other personal services,Manufacture of cement,185.390777
Other personal services,Manufacture of concrete and concrete products,76.72158652
Other personal services,Manufacture of basic iron and steel,761.1795496
Other personal services,Manufacture of other basic metals,441.697268
Other personal services,Manufacture of fabricated metal products,266.8860295
Other personal services,Manufacture of industrial and domestic machinery and equipment,374.0411672
Other personal services,Manufacture of electric and electronic machinery and equipment,166.1190347
Other personal services,Manufacture of transport equipment,26.53889606
Other personal services,Manufacture of furniture,147.1293956
Other personal services,Repair and installation of machinery and other manufacturing,78.24130248
Other personal services,Electric power generation,1831.407249
Other personal services,Transmission of electric power,235.2911844
Other personal services,Distribution of electric power,904.7673434
Other personal services,Gas and steam manufacture and supply,123.5558747
Other personal services,Water collection treatment and supply,77.44865887
Other personal services,Waste collection and recycling activities,1626.48131
Other personal services,Construction of residential buildings,189.1751981
Other personal services,Construction of non-residential buildings,109.8551272
Other personal services,Civil engineering,731.0134728
Other personal services,Specialized construction activities,205.0286673
Other personal services,Wholesale and retail trade of motor vehicles,260.9984772
Other personal services,Wholesale trade,2541.286374
Other personal services,Retail trade,1180.965395
Other personal services,Accommodation,9257.328572
Other personal services,Food and beverage service activities,10550.62931
Other personal services,Transport via railways,263.0610816
Other personal services,Other passenger land transport,1752.310715
Other personal services,Freight transport by road,2341.935143
Other personal services,Transport via pipeline,131.1800151
Other personal services,Water transport,234.1414255
Other personal services,Air transport,2904.66238
Other personal services,Warehousing,31.89888845
Other personal services,Support activities for land transport,78.07697413
Other personal services,Other support activities for transport,766.4323588
Other personal services,Postal and courier activities,31.09044712
Other personal services,Wireless telecommunications,748.1358257
Other personal services,Wired telecommunications,1176.278878
Other personal services,Other telecommunications activities,1288.941839
Other personal services,Information services activities,1361.034577
Other personal services,Publishing activities,1630.896349
Other personal services,Banking monetary intermediation,1123.386859
Other personal services,Insurance activities,258.7535836
Other personal services,Auxiliary financial activities,383.0434721
Other personal services,Real estate activities,407.3836483
Other personal services,Housing services,0
Other personal services,Legal and accounting services,79.44650228
Other personal services,Architectural and engineering services,986.2758873
Other personal services,Other professional activities,842.5699327
Other personal services,Rental and leasing activities,1833.155687
Other personal services,Business support activities,3670.976513
Other personal services,Public administration,1917.261305
Other personal services,Public education,297.1873432
Other personal services,Private education,374.1518112
Other personal services,Public health activities,22590.48205
Other personal services,Private health activities,11840.38031
Other personal services,Activities of organizations,71.41895096
Other personal services,Artistic and entertainment activities,1512.202122
Other personal services,Other personal services,246.5314007"""
  |> fromCSV
```
