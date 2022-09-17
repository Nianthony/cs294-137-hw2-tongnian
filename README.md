# HW2: Building Unity Game in Simulated AR

In this homework I build a simple AR Game, Whack a Mole. Here are the reference, features of my game and google drive link for my unity package and introduction video  

## Reference

After I decided to make a whack a mole game. I first searched those free 3D model on internet. They don't have a good visual performance. And I found this template in the unity asset store. 

https://assetstore.unity.com/packages/templates/whack-a-mole-82155

I used the prefabs (moles, trees and stones), font, audio and part of the animation to set up my unity scene. I took part of the codes in the template as my reference or tutorial for coding my own part. I rewriting the judgement condition in the game mechanism for it works different in the ARsimulation environment becuase those Raycast funtion cannot be simply implement in the simulation environment.


## Link

I used the .gitignore file and tried to push my unity files to my repo. But I got lot of troubles in the process, like the name of my files are too long and there are lot of .meta fils doesn't include in the .gitignore template. I consdier it would be very inconvenient to view the incomplete unity engineering documents. So I export the unity package and put it in my google drive link. The video is also in my google drive.

https://drive.google.com/drive/folders/1WTnmnm6wMBOO2kNQbf5k3FHDkrtFZsWe?usp=sharing


## Features of my game

### 1.Camera following when moving the board.
Before the game start, you will need a broad view of all the 3 panels for choosing where to set your gameboard. Upon the game begin, you need to zoom in on the camera for a better gaming experience. So I code this camera following script. After you have already placed the game board, camera will zoom in. Otherwise if you are placing the game board, camera will zoom out to the original position. This script also coordinates with the camera shake script.  
```C++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ARFollowTarget : MonoBehaviour
{
    // follow parameter 
    public float distanceAway = 10;           
    public float distanceUp = 2;             
    public float smooth = 3; 

    public GameObject gameBoard;
    Transform follow;

    // store the original position
    private Vector3 targetPosition, originalPosition;
    private Quaternion originalRotation;

    public bool placed = true;

    void Start()
    {
        follow = gameBoard.transform;
        originalPosition = this.transform.localPosition;
        originalRotation = this.transform.localRotation;
    }

    void Update()
    {
        // follow the board
        if (gameBoard.active && placed)
        {
            targetPosition = follow.position + Vector3.up * distanceUp - follow.forward * distanceAway;
            transform.position = Vector3.Lerp(transform.position, targetPosition, Time.deltaTime * smooth);
            transform.LookAt(follow);
        }      
        // back to the original position
        else if (gameBoard.active && !placed)
        {
            transform.position = Vector3.Lerp(transform.position, originalPosition, Time.deltaTime * smooth);
            transform.rotation = Quaternion.Lerp(transform.rotation, originalRotation, Time.deltaTime * smooth);
        }
    }

    public void moveBoard( )
    {
        placed = false;

    }
}

![camera follow.png](https://github.com/Nianthony/cs294-137-hw2-tongnian/blob/aa22f7a0d2560ee0a0b4c70ac0738ce961985e67/image/camera%20follow.png)


### 2.OnTouch function for determine the state of mole
You 

![on touch.png](https://github.com/Nianthony/cs294-137-hw2-tongnian/blob/aaa7357f924389e5427a1fd806bcf78782585799/image/on%20touch.png)



### 3.Camera shaking when the hammer hitting the mole
You 

![camera shake.png](https://github.com/Nianthony/cs294-137-hw2-tongnian/blob/aaa7357f924389e5427a1fd806bcf78782585799/image/camera%20shake.png)

