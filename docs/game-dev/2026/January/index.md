---
title: January 1 - Mobile Shooter
---

First game of my [2026 challenge](../../../../blog/a-fresh-start), and I'm already breaking the rules a bit in that I didn't start with a design document. In my defense, I didn't really have the challenge in mind before I started building, so I'm going to excuse myself this one time. 

December 2025 I got the hankering to dust off Unity and take another swing at making a game. I started working on something I'm excited about, but quickly realized that my knowledge with the Unity was a bit dated. So I figured I'd shelve that project for now and give my self something simpler as a warm up. 

The project in question is a style of mobile game I've seen a couple of adds for recently (Last War: Survival being a recent example), where the player controls a squad of soldiers shooting endlessly at a wave of zombies walking towards the bottom of the screen. There's also power-ups that can add squad members, or increase the firepower of the squad. Every so often a more robust zombie appears, requiring far more shots than the rest. The challenge is in keeping the horde suppressed while also getting strong enough to deal with larger zombies. 

![](lastwarimage.jpg)

Admittedly I've never played this game before, but I feel like I understand the concept from the ads I've seen. So, I started prototyping using some basic 3D objects in Unity. After not too long I had something that could be described as "playable".

![](mobile-game-1.png)


# Object Pooling

I've instantiated my fair share of objects in Unity before. However, I've never designed a game to create objects at a large scale. After some cursory searching it looks like a common approach here is something called Object Pooling. 

The concept, as I understand it anyways, is fairly simple. Instantiating and destroying objects in a game engine has some processing overhead. So instead of creating and destroying hundreds of objects I should instead start the game with a large number of them and reuse them at every opportunity. 

To give a more mundane example, let's say you're responsible for office supplies at a company. At the start of the day, all 20 employees ask you for a pen to use during the day. For each employee, you grab a package of pens, take one out, and hand it to the employee. At the end of the day, you collect each pen and toss them into the trash. This works, and gets everyone a pen to use. 

However, the approach is wasteful. Not only can you reuse the pens, saving on the cost of buying more, you're also eating up time by having employees ask you for a pen. You know there's 20 people, so why not just grab a mug, place 20 pens in it, and leave the mug somewhere the employees can find it? If they want to return the pen they know where to put it, and someone else can reuse a returned pen. Additionally, this approach scales well. If the company highers ten more employees, put that many more pens in the mug. 

It's a silly analogy, but demonstrates what Object Pooling is. For Mobile Shooter game, I have an object pool for projectiles, zombies, and power-ups. When I would "destroy" an object, such as when a projectile collides with a zombie, we instead deactivate the objects. Then when we need to make more, the object pool will activate one of the dormant objects.

Instead of calling `Destroy()` on something, we can do the following:

```cpp
    public void Release(GameObject obj)
    {
        obj.SetActive(false);
        obj.transform.SetParent(transform);
        
        // Add the object to the end of the pool
        _inactive.Enqueue(obj);
    }
```

When it's time to make another, we can do the following:

```cpp
    public GameObject Get(Vector3 position, Quaternion rotation)
    {
        GameObject obj;
        if (_inactive.Count > 0)
        {
            // Remove and return the object at the start of the queue
            obj = _inactive.Dequeue();
        }
        else
        {
            // If we've outgrown the queue, we can make more
            if (!canGrow) return null;
            obj = CreateNew();
        }

        // Reset the object to some initial state and then make it active
        obj.transform.SetPositionAndRotation(position, rotation);
        obj.SetActive(true);

        return obj;
    }
```

There's a little bit more to the actual implementation, but that's enough to get the point across for this post. 

# What's Next For the Project?

Currently the game has:

- Enemies
- Ways to get more soldiers
- An end state (either a zombie making it to the end of the screen, or the zombies eating all of the player's soldiers)
- "Big zombies", which periodically spawn and force the player to spend some time shooting them. 

The game is lacking a few things:
- A win condition - the game goes on until the player looses
- There could be some better animations for things
- Possibly some better 3D models, although I do worry about performance taking a hit if I just grab a zombie from the asset store
- A title screen
- An actual title ("Mobile Shooter" is obviously a placeholder)
- An ending screen ("You Won" or "Game Over")
- Any sort of sound. 


while I don't plan on turning this into a full game, I think I should address the above items before saying I completed this month's effort. 

See you in the next one. 