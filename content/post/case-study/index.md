---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "A Case Study in Object Oriented Programming: Catan in Python"
subtitle: "Revisiting OOP concepts in a familiar setting"
summary: "An attempt at explaining why Object Oriented Programming is touted to be the saviour of our times"
authors: 
- admin
tags: 
- Academic
categories: 
- Blog
date: 2021-03-06T19:47:14-05:00
lastmod: 2021-03-06T19:47:14-05:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: ["internal-project"]
---
# A Case Study in Object Oriented Programming: Catan in Python

Object oriented programming can be fun and rewarding. When you really get into it, you almost get the feeling that the code _builds itself_ after a point. When you're taught Object Oriented Programming in a Software Engineering context at a college level computer science course, it's often placed early on, _right after_ you've been taught a new language (or right after you're taught programming in general). This sometimes makes it hard to really appreciate what value it adds to your software engineering toolkit. 

I was recently playing a game of _[Catan](https://www.catan.com/)_ and found the interaction between settlements, cities, roads and the hexagonal tiles particularly interesting (and no, I _wasn't_ doing this just because I was losing, why do you ask?). Trying to model these interaction and properties in code seemed like a tasty challenge, one that I can easily sink a weekend into. If I wanted to show the usefulness of object oriented programming, (Although, I should really pick a board game that didn't have a 20-page [rule book](https://www.catan.com/service/game-rules)). I'm not going to try and build an entire working game engine for this blog post; just a few classes and interactions. By the end of this blog post, I hope to have answered that itching question you've always wanted to ask your high school programming teacher or your Intro to Software Engineering professor: _What's the big deal about Object Oriented Programming Anyway?_

## Catan Brush-Up
If you're rusty in your _Catan_, [this video](https://www.youtube.com/watch?v=hY2KnwfEBkA) does a pretty good job of explaining the rules. Each turn basically boils down to this:

## OOP's core tenants
Object Oriented programming has four main principles: Encapsulation, Abstraction, Inheritance and Polymorphism. I'd say that if you have a good intuitive understanding of the first three, you can build a strong understanding of and appreciation for OOP.

### Abstraction

First, I'll model the hexagonal tiles that are synonymous with _Catan_. There are 6 different types of tiles, and each one generates a different resource: _Pastures_ generate _Wool_, _Forests_ generate _Wood_, _Hills_ generate _Bricks_, _Mountains_ generate _Ore_, _Fields_ generate _Grain_ and _Deserts_ generate ... well, nothing. Writing down definitions for all of these in this blog post is a bit pointless, so I am going to focus on two: _Forests_ and _Pastures_.


```python
class Tile(abc.ABC):
    @abc.abstractmethod
    def name(self):
        pass

    @abc.abstractmethod
    def generate_resource(self):
        pass

    @abc.abstractmethod
    def short_name(self):
        pass

class Forest(Tile):
    def name(self):
        return "Forest"

    def generate_resource(self):
        return Wood()

    def short_name(self):
        return "fo"

class Pasture(Tile):
    def name(self):
        print("Pasture")

    def generate_resource(self):
        return Wool()

    def short_name(self):
        return "pa"
    
class GameTile:
    def __init__(self, number_label, tile, points, position):
        self.number_label = number_label
        self.tile = tile
        self.points = points
        self.position = position
```

The eagle-eyed among you would have noticed that I haven't defined `Wood` and `Wool` anywhere yet. Good catch! Here they are:


```python
class ResourceCard(abc.ABC):
    @abc.abstractmethod
    def name(self):
        pass

class Wood(ResourceCard):
    def name(self):
        return "Wood"

class Wool(ResourceCard):
    def name(self):
        return "Wool"
```

Its nothing fancy or complex, all it does is make each of the resource card objects human readable by returning a name. These definitions let me do something like this somewhere in my game engine code:


```python
def game_loop():

#
# ... complex engine code
#

    # get the active player to roll the dice
    roll = player.roll_dice(dice)
    
    # get the GameTiles that have the label associated with current roll
    active_tiles = game_tiles[roll]
    
    # for each of the tiles that matched
    for game_tile in active_tiles:
        # for each point in the hexagonal tile
        for point in game_tile.points:
            # If the point has a setllement or a city
            if point.abode is not None:
                # get the owner
                reciever = point.abode.owner
                # Give him the resource Card!
                reciever.add_resource(game_tile.tile.generate_resource())
                
#
# ... complex engine code
#
```

There might be a lot going on there but focus on this specific line: `reciever.get_resource(game_tile.tile.generate_resource())`. That's the game engine calling the `generate_resource()` method I wrote in the definition above. Since I 'hid' away the logic to actually generate the resource into the `Tile` classes, I don't have to manually check the type of each tile when I hand out the resource cards to the player!

This idea of 'hiding away' code and logic that is not useful to the _current_ situation is called ***Abstraction***. Another intuitive way to think of abstraction is using code as _building blocks_: I used the implementation of the tiles' `generate_resource()` method to _build upon_ for the game engine logic. Also, the `Forest` class _itself_ builds upon the code defined in the `Wood` class.

### Encapsulation

I hinted at the existence of a `Player` class in the game engine code above. Let's look at it now:


```python
class Player:
    def __init__(self, color):
        self.color = color
        self.victory_points = 0
        self.resource_cards = ResourceCardDeck()
        self.development_cards = []
        
    def add_resource(self, resource_card):
        """
        Adds the given resource into the player's hand
        """
        self.resource_cards.add_card(resource_card)
        
    def accept_trade(self, incoming_trade, requested_trade):
        """
        Adds the cards from the incoming trade to the player's hand and 
        removes those from the requested trade
        
        both incoming_trade and requested_trade are dictionaries where the key is the card type
        and the value is the number of cards of that type.
        """
        for card in requested_trade:
            number_of_cards = requested_trade[card]
            self.resource_cards.remove_cards(card, number_of_cards)
        for card in incoming_trade:
            number_of_cards = incoming_trade[card]
            self.resource_cards.add_cards(card, number_of_cards)
    
    def add_development_card(self, card):
        """
        Adds the given development card into the players hand
        """
        self.development_cards.append(card)
```

> Note: The implementation of adding and removing cards from the resource cards in the players hand is not simple and not relevant here, so I, wait for it, _abstracted_ it away :)

Each turn, a player could:
1. Get resource cards if the die roll is in their favor.
2. Use their resource cards to buy a development card.
3. Trade resource cards with another player.

For the sake of simplicity, I am going to stick to modeling just these three actions. Each of these actions are handled in each of the three methods in the `Player` class. Each of those methods in turn, manipulate the resource cards or development cards in the player's hand.

What we've done here by clubbing together a classes _data_ (the player's hand) and _functions that affect the data_ (the three actions above) into one class is called ***Encapsulation***. It also makes it so that a player's hand can only be modified by actions that happen to _that_ player. (In a legal game, you wouldn't have players stealing each others cards!). Encapsulation 'protects' the classes data from code outside the class.


### Inheritance

No version of _Catan_ is complete without the all-important Settlements and Cities. Here is what they look like in my code:


```python
class Building(abc.ABC):
    
    def __init__(self, owner, name):
        self.owner = owner
        self.name = name

    def description(self):
        return "%s owned by %s" % (self.name, self.owner)

class Abode(Building):
    def __init__(self, owner, name, victory_points):
        super.__init__(owner, name)
        self.victory_points = victory_points

class Settlement(Abode):
    def __init__(self, owner):
        super.__init__(self, owner, "Settlement", 1)

class City(Abode):
    def __init__(self, owner):
        super.__init__(self, owner, "City", 2)

class Road(Building):
    def __init__(self, owner):
        super.__init__(self, owner, "Road")

class Point:
    def __init__(self, abode, position, n1, n2, n3):
        self.abode = abode
        self.n1 = n1
        self.n2 = n2
        self.n3 = n3
```

What I've done here is that I've leverage the _is-a_ relationship some of these buildings have. For example, Settlements and Cities are similar in that the count towards a player's victory points. I called these victory point generating buildings _Abodes_. _Cities_ and _Settlements_ are _Abodes_; _Roads_ are not. They are all still buildings though, and can be owned by players. 

I've modeled these hierarchical relationships using ***Inheritance***. Inheritance often goes hand-in-hand with abstraction. You can see it in action here: I abstracted away code that is specific to a city into the `City` class. 

## Conclusion

What's covered in this blog post is just the tip of the iceberg though! There is still quite a lot I didn't cover: Delegation, Polymorphism, Dynamic Dispatch (We did do a fair bit of this in this post ,though) and Composition. However, my intention with this post was to hopefully help you build an intuitive understanding of _why_ OOP is preached so much, not to teach it outright.

Hopefully, you saw what I meant when I meant when I talked about the code building itself if you approach your problem with strong Object Oriented Programming ideals. I'm hoping that reading this blog post encourages you to try and model your favorite board game! (If its _Catan_, I'm open to any improvements you might suggest to what I've written.)

The demo code I used in this post can be found on my [GitHub](https://github.com/vigneshRajakumar/Catan).
