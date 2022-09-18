# HW2: Building Unity Game in Simulated AR

In this homework, I build a simple AR Game, Whack a Mole. Here is the reference, features of my game, and google drive link for my unity package and introduction video.  

## Reference

After I decided to make a whack-a-mole game. I first searched for those free 3D models on the internet. They don't have a good visual performance. And I found this template in the unity asset store. 

https://assetstore.unity.com/packages/templates/whack-a-mole-82155

I used the prefabs (moles, trees, and stones), font, audio, and part of the animation to set up my unity scene. I took part of the codes in the template as my reference or tutorial for coding my own part. I rewrote the judgment condition in the game mechanism because it works differently in the AR simulation environment because those Ray cast functions cannot be simply implemented in the simulation environment.


## Link

I used the .gitignore file and tried to push my unity files to my repo. But I got a lot of troubles in the process, like the name of my files are too long and there are a lot of .meta files doesn't include in the .gitignore template. I consider it would be very inconvenient to view the incomplete unity engineering documents. So I export the unity package and put it in my google drive link. The video is also in my google drive.

https://drive.google.com/drive/folders/1WTnmnm6wMBOO2kNQbf5k3FHDkrtFZsWe?usp=sharing


## Features of my game

### 1.Camera following when moving the board.
Before the game starts, you will need a broad view of all the 3 panels to choose where to set your gameboard. Upon the game’s beginning, you need to zoom in on the camera for a better gaming experience. So, I code this camera following the script. After you have already placed the game board, the camera will zoom in. Otherwise, if you are placing the game board, the camera will zoom out to the original position. This script also coordinates with the camera shake script.   
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

```

![camera follow.png](https://github.com/Nianthony/cs294-137-hw2-tongnian/blob/aa22f7a0d2560ee0a0b4c70ac0738ce961985e67/image/camera%20follow.png)


### 2.OnTouch function for determine the state of mole
As the original Unity functions work differently in the simulation environment. Instead of determining whether the collider volumes of the moles and hammer collide, I choose the onTouch function to determine whether your mouse clicks the right mole in time. If the mole is clicked by your mouse, then the hammer will move directly over the mole and finish a rapid vertical fall and ascent in 0.25 seconds. Meanwhile, the score will add. 
```C++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MoleController : MonoBehaviour, OnTouch3D
{

	private float moveSpeed = 0.1f;
	private float waitTime = 1.0f;

	private float TOP = 0.44f;
	private float BOTTOM = 0.35f;
	private float tmpTime = 0;

	enum State
	{
		UNDER_GROUND,
		UP,
		ON_GROUND,
		DOWN,
		HIT,
	}
	State state;

	public void OnTouch()
	{
		bool isHit = Hit();
		// if hit the mole, score, hummer and effect
        if (isHit)
        {
			Debug.Log("Hit U: " + gameObject.transform.position);
			/*GameObject hummer = GameObject.Find("Hummer");
			hummer.GetComponent<HummerController>().hitPosition = gameObject.transform.position;
			hummer.GetComponent<HummerController>().hit = true;*/
			HummerController.hitPosition = gameObject.transform.position;
			HummerController.hit = true;
		}
    }

	//connect with mole controller
	public void Up()
	{
		gameObject.SetActive(true);
		if (state == State.UNDER_GROUND)
			state = State.UP;
	}

	public bool Hit()
	{
		// if mole is under ground, never hit
		if (state == State.UNDER_GROUND)
			return false;

		// hit to bottom
		transform.position = new Vector3(transform.position.x, BOTTOM, transform.position.z);
		state = State.UNDER_GROUND;
		return true;
	}

	void Start()
	{
		// all set to the bottom
		state = State.UNDER_GROUND;
	}

	void Update()
	{
		// show up
		if (state == State.UP)
		{
			transform.Translate(0, moveSpeed, 0);
			if (transform.position.y > TOP)
			{
				transform.position = new Vector3(transform.position.x, TOP, transform.position.z);
				state = State.ON_GROUND;
				tmpTime = 0;
			}
		}

		// based on time to determine go bottom or not
		else if (state == State.ON_GROUND)
		{
			tmpTime += Time.deltaTime;
			if (tmpTime > waitTime)
				state = State.DOWN;
		}

		// go bottom
		else if (state == State.DOWN)
		{
			transform.Translate(0, -moveSpeed, 0);
			if (transform.position.y < BOTTOM)
			{
				transform.position = new Vector3(transform.position.x, BOTTOM, transform.position.z);
				state = State.UNDER_GROUND;
				gameObject.SetActive(false);
			}
		}
	}

}

```

![on touch.png](https://github.com/Nianthony/cs294-137-hw2-tongnian/blob/aaa7357f924389e5427a1fd806bcf78782585799/image/on%20touch.png)



### 3.Camera shaking when the hammer hitting the mole
To improve the immersive game experiences, I added the prefab of explosion in my reference asset to the scripts. When the hammer hit the mole, it will trigger the animation of an explosion. At the same time, the camera will make a quick offset to simulate the effect of vibration on your perspective.
```C++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CameraShaker : MonoBehaviour {

	Vector3 defaultPos;
	public float MAGNITUDE = 0.1f;

	public void Shake()
	{		
		StartCoroutine (Shake_Resilience());
	}

	IEnumerator Shake_Resilience()
	{
		for (int i = 0; i <= 360; i += 45) 
		{
			transform.position = new Vector3 (defaultPos.x, defaultPos.y + MAGNITUDE*Mathf.Sin (i * Mathf.Deg2Rad), defaultPos.z);
			yield return null;
		}
	}

	public void defaultPosition()
    {
		defaultPos = transform.position;
	}
}

```

![camera shake.png](https://github.com/Nianthony/cs294-137-hw2-tongnian/blob/aaa7357f924389e5427a1fd806bcf78782585799/image/camera%20shake.png)

