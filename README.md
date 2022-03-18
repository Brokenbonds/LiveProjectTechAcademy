# LiveProjectTechAcademy <br>
At the end of my Unity/C# programing module, I was tasked with a 2-week sprint to create an old arcade game in which I chose Lunar Lander. I was challenged not only with the coding stories that were presented to me but also with getting some good practice in a repository. After creating a clone from a Microsoft Azure Repository that held a Unity Project in which all other students were creating their programs as well, I was set to task. Some of the bigger challenges for me were procedural map creation and a fluid audio system. I called my project Ti.Tian Lander, which is a play off the periodic table element Titanium.

# Title Screen <br>
The title screen was a simple image with a button to get you started

# Tutorial page <br>
This page runs through the simple controls and objective in the game

# The Level <br>
Currently, there is only one level that repeats but due to the procedural map, the level is never the same. Increasing the playability of my simple game. You can find the code [here](#mesh-code) In the Awake method, I used a Vector3 List to create vertices for my Mesh which was the foundation of my map. From there, I turned them into a Vector 2 array and used this to ensure everything was on the same Z scale. Using another Array I created groups of 3 integers that would represent triangles for my mesh. I then tackled the normals of my mesh by creating a brand new Vector 3 List, all though I could have most likely just used the Mesh.RecalculateNormals to achieve a similar effect with less control.

Once I had accomplished this I started working on instantiating a [single sprite](#single-sprite-placement) along the distance of my Vector2, I did this by grabbing the overall distance and creating a random number between 0 to max distance. Once I had this I was halfway there, I then had to figure out between which Vector 2s this number lay. I created an array that held distance from A-B, B-C, C-D, etc. then by running through this array I checked to see if the previous distance + the currant distance was bigger than my random distance. I was now able to determine which points I was landing between. Using a Lerp I was then able to put in these two Vector2 points and then divided my max distance and my random point after subtracting the distance between 0 and the closest point and voilà.

Now that I had this and new my therory was working correctly, I started to go bigger by [populating my map](#Single-Sprite-Placement) with a large amount of sprites that was dictated by 2 different arrays.

After I had created this I worked on other asspects of the game to see if I coudl bring it to pollish before the end of my 2 weeks. Such as Audio the blended between Scences, Rocket sounds that only played with the player pressed a key, and an explosion sound when the rocket hit the ground. I hope to expand on this project more in the future and post it to itch.oi.

# Mesh Code
```
 private void Awake()
    {
        Mesh mesh = new Mesh();
        mesh.name = "Ground";

        /*this is a list of vector 3 points which I will use as vertices to make a square. Positions are listed as followed.
         * 1  2  3  4  5  6  7
         * 0 13 12 11 10  9  8
         */
        List<Vector3> points = new List<Vector3>() {
            new Vector3(-9, -5),//this is the point in 3d space the vert is placed
            new Vector3(-9, Random.Range(-5,4)),
            new Vector3(-6, Random.Range(-4,4)),
            new Vector3(-3, Random.Range(-4,4)),
            new Vector3(0, Random.Range(-4,4)),
            new Vector3(3, Random.Range(-4,4)),
            new Vector3(6, Random.Range(-4,4)),
            new Vector3(9, Random.Range(-5,4)),
            new Vector3(9, -5),// to stop verts from overlaping, I'm placing a bottom vertex I can connect to.
            new Vector3(6, -5),
            new Vector3(3, -5),
            new Vector3(0, -5),
            new Vector3(-3, -5),
            new Vector3(-6, -5),

        };
        /*Note that if you make a cube you can create 8 verts to creaet a cube but it will not have hard edges
         * to create hard edges you must create 3 verts on each corner, 24 in total.
         */


        // convert to list of Vector 2s for Polygon Collider
        List<Vector2> coPoints = new List<Vector2>();
        for(int i = 0; i<points.Count; i++)
        {
            coPoints.Add(points[i]);
        }
        Vector2[] coPointsArray = new Vector2[coPoints.Count];
        coPointsArray = coPoints.ToArray();


        //this array holds the direction of how to build a triangle. mesh should be built clockwise.
        //each int is a position in our List of Vector3s
        //a set of 3 creates a triangle
        int[] triIndices = new int[]
        {
            0,1,13,
            13,1,2,
            12,13,2,
            3,12,2,
            11,12,3,
            4,11,3,
            4,10,11,
            4,5,10,
            5,9,10,
            5,6,9,
            6,8,9,
            7,8,6,
        };

        // same as the verts these are just normals. They need Vector3s to show which way the faceing.
        List<Vector3> normals = new List<Vector3>()
        {
            new Vector3(0,0,-1), //this is the direction the normal of that verticy is looking
            new Vector3(0,0,-1),

            new Vector3(0,0,-1),
            new Vector3(0,0,-1),

            new Vector3(0,0,-1),
            new Vector3(0,0,-1),

            new Vector3(0,0,-1),
            new Vector3(0,0,-1),

            new Vector3(0,0,-1),
            new Vector3(0,0,-1),

            new Vector3(0,0,-1),
            new Vector3(0,0,-1),

            new Vector3(0,0,-1),
            new Vector3(0,0,-1),

        };


        mesh.SetVertices(points); //setVertices takes a list, vertices takes an array
        mesh.triangles = triIndices; // this sets the triangles.

        mesh.SetNormals(normals); // this uses our list of normals to creat the normals instead.
        //mesh.RecalculateNormals(); // this will have unity recalculate the normals of our gemoitry for us.

        GetComponent<MeshFilter>().sharedMesh = mesh;//calling the shared mesh part of MeshFilter component. which allows the meshfilter to build the mesh and we can see it.

        PolygonCollider2D polyCollider = GetComponent<PolygonCollider2D>();

        polyCollider.pathCount = 0;
        polyCollider.SetPath(0, coPointsArray);

        
        List<Vector2> pointsNeeded = new List<Vector2>(); //wrote this part just in case I wish to put in more map Vertices later
        for (int i = 1; i <= (coPoints.Count)/2; i++)//the math for the < variable only works with a square that has equal points on either side.
        {
            pointsNeeded.Add(coPoints[i]);
        }

        SpaceSationPlacement(square, pointsNeeded);
        MapSpritePopulation(planetSprites, planetSpritesPopulationAmout, pointsNeeded);
       
    }
```

# Single Sprite Placement
```
void SpaceSationPlacement(GameObject obj, List<Vector2> points)
    {
        //this first section goes through the Vector 2 array and gets the distance between points 1-7(or higher)
        //the reason for this is because of the direction I laid out my points.
        float dis = 0;
        for (int i = 0; i < points.Count-1; i++)//this should only run 5 times, for now.
        {
            dis = dis + Vector2.Distance(points[i], points[i+1]);
            
        }


        float spawnPositoin = Random.Range(3, dis-3);//get a random distance from the max distance.
        

        float[] individualeDistances = new float[points.Count];// need to populate this array with each distance 1-2,2-3,3-4,etc.
        for (int i = 0; i < points.Count-1; i++)//use this for loop to populate above array
        {
            dis = 0;
            dis = Vector2.Distance(points[i], points[i + 1]);
            individualeDistances[i] = dis;
        }

        float disCheck = 0;

        for (int i = 0; i < individualeDistances.Length; i++)
        {

            if (disCheck + individualeDistances[i] > spawnPositoin) // this checkes everything else 2-3 all the way to 6-7(or higher if i want) instantiates the object.
            {
               
                float equalsThis = (spawnPositoin -disCheck) / individualeDistances[i];
                GameObject.Instantiate(obj, Vector2.Lerp(points[i], points[i + 1], equalsThis), Quaternion.Euler(0, 0, 0));
                //this will do the same as before but subtract the max disCheck from spawnPosition so it doesn't go to far.
                return;
            }
            else // if the above code fales, obvisouly my spawnPosition is farther so I want to add the current distance to my disCheck for the next loop
            {
                disCheck += individualeDistances[i];
            }

        }


    }
```
# Array Sprite Populator
```
 void MapSpritePopulation(GameObject[] obj,List<int> amountToSpawn, List<Vector2> points)
        /* GameObject array holds the sprites
         * int array hold the quantaties for each sprite
         * Vector2 List holds the points I will be spawning across
         */
    {

        //this first section figures out how many sections on the map there are and their distances.
        float[] individualeDistances = new float[points.Count];// need to populate this array with each distance 1-2,2-3,3-4,etc.

        for (int i = 0; i < points.Count - 1; i++)//use this for loop to populate above array
        {
            float disSmall = 0;
            disSmall = Vector2.Distance(points[i], points[i + 1]);
            individualeDistances[i] = disSmall;
        }

        //this section goes through the Vector 2 arrays and gets the distance between points 1-7(or higher)
        //the reason for this is because of the direction I laid out my points.
        float overAllDistance = 0;
        for (int i = 0; i < points.Count - 1; i++)//this should only run 5 times, for now.
        {
            overAllDistance = overAllDistance + Vector2.Distance(points[i], points[i + 1]);

        }


        for (int k = 0; k < individualeDistances.Length; k++)// this is looking at how far we have instantiated on the Vector2s
        {
            for (int i = 0; i < obj.Length; i++)// this looks at what object we are spawning multipules of
            {
                for (int j = 1; j <= amountToSpawn[i] / (individualeDistances.Length - 1); j++)// this looks to see if the amount of objects has been reached. Position of the number in the array should corilate to the number position of the object being instantiated.
                {
                    //put in list that will store the x floats and check to see if there is a same number before exicuteing the instantiate functions.

                    float spawnPositoin = Random.Range(0, individualeDistances[k])/individualeDistances[k];//get a random distance from the max distance for that individuale distance.

                    GameObject.Instantiate(obj[i], Vector2.Lerp(points[k], points[k + 1], spawnPositoin), Quaternion.Euler(0, 0, 0));


                }
            }
        }
    }
```
