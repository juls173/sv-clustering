# Street View Photosphere Clustering
Scalable. reproducible pipeline for clustering Google Street View photospheres based on visual similarity.
I used the [StreetCLIP transformer](https://huggingface.co/geolocal/StreetCLIP) to embed images, then K-means to cluster them. Uses https://map-generator.vercel.app/ to generate the locations and the Google Street View API to get the panoramas.
The main goal of this project was to see what categories of visually similar locations a model pre-trained on Street View images would come up with and how these differed from my own and other Geoguessr players' categories. This can also provide pre-made categories or chunks of visually similar locations to learn from.

### Steps (more details/code in gsvembed.ipynb notebook): 
1. First, we need to obtain a set of coordinates corresponding to Google Street View coordinates. https://map-generator.vercel.app/ or https://github.com/slashP/Vali are perfect for this and allow you to filter locations and select the area in which you'd like to query coordinates for, e.g., allowing you to choose a specific country, subdivision, or hand-drawn polygon. 
MapGenerator is simpler, while Vali allows for less naive location generation and more customizability.
In MapGenerator, select a country or study area (I selected Chile), hand-draw a study area, or import a custom layer for more precise boundaries.
You can also choose the number of locations you want to generate, I choose 1,000
2. We can parse through the generated JSON and request images from the [Google Street View API](https://developers.google.com/maps/documentation/streetview/request-streetview) using the coordinates generated. I requested 4 images per location, facing 0, 90, 180, and 270 degrees, then stitched these images together to get a 360-degree panorama. I downloaded all these images. 
3. I then used the StreetCLIP transformer to generate embeddings for each stitched panorama, applied L2 normalization to these embeddings, saved them to an embeddings folder, and created a metadata file that tracks which embeddings corresponded to which panoramas.
4. Next, I used the sci-kit learn library to use K-Means clustering to group locations that are closer together in the embedding space. I also experimented with using PCA for dimensionality reduction and differing numbers of clusters.  
5. Lastly, we can write the generated clusters as metadata to each location in the original JSON as a map-making.app tag.

### Results
Here are maps of the resulting clusters
18 clusters: <img src="https://github.com/user-attachments/assets/28a8a1f7-a119-4189-8f33-56d8c59de1d6" width="480"> 5 clusters:<img src="https://github.com/user-attachments/assets/22b2e2fc-dc82-4898-8d2d-8d1efab9ca0e" height="480">
The clusters generally have a lot of overlap with other clusters but are still fairly geographically localized. In the 5-cluster example, there are regions where 90% of the locations ended up in a single cluster, such as the Atacama desert and Aysén, which is a really good sign. 
Within the Atacama desert, the only locations that didn't end up in the main cluster were urban or heavily vegetated, which is what we'd intuitively expect to see, as these locations are more visually similar/could be confused with other parts of Chile. 

Here are two locations that ended up in the same cluster but, in reality, are 2,100 km apart
![image](https://github.com/user-attachments/assets/875c2030-18fc-41d8-af9f-3a5c001bd998)
![image](https://github.com/user-attachments/assets/18eb6c82-82d3-46bf-9168-f629ca43156c)
The results are fairly interesting, but I probably shouldn't anthropomorphize them.

Also, Chile is probably one of the least valuable countries to which this pipeline can be applied--Chile already has pretty distinct and clear biome, landscape, and visual categories--I just used it as a proof of concept to see if the generated clusters matched highly intuitive ones. 
There are some better use cases: 
1. Applying this auto-clustering on coverage-dense, decently-sized areas that don't have clearly chunked visual differences but are still visually heterogeneous (Texas, Rajasthan, and Uttar Pradesh come to mind) would provide useful premade mental categories to study from.
2. Significantly increasing the number of clusters or locations could result in localized road-specific or coverage-specific clusters, pointing out easily learnable/distinct roads/areas (e.g., the coverage on a particular road might be cloudy only on a specific stretch).

### Folder structure/recommended setup:
This is how I set up the folder structure for things not included in this GitHub repository
chile_tagged/
  /[output json files with cluster tags]
embeddings/
  /embedding_metadata.json
  /[1000 embedding .pt files, one for every location]
panoramas/
  /location_[i 0 -> 999]/
    /heading_0.jpg
    /heading_90.jpg
    /heading_180.jpg
    /heading_270.jpg
venv/ (recommended to avoid package dependency conflicts with global environment)

Feel free to import the output JSON files in chile_tagged into map-making.app and explore them yourself!

Fully credit to Sean(Ethereal) for the idea!
