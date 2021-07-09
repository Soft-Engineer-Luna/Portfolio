---
menu: Paper2D AutoTiling
tab: false
parent: Engineering.md
weight: 2
---

# intergrating a auto tiling system in too the Paper2D Plugin

I decided to fill the gap in the paper 2d Plugin, it lacked the ability to quickly and easily 
create maps. So I took it upon myself to do the required research and integrate an autotiling 
system through bit masking patterns. After research I choose to implement the pattern used by the most popular 2D rpg engine RPGMaker.

<br/><br/>
![Tiling Pattern](Media/AutoTilingPattern.jpg?raw=true "Tiling pattern")
###### image from RPGMaker forum same pattern used in the Tiling System i built
<br/><br/> 
I choose to support different tile types and eight directional auto tiling to get a much more accurate and nicer looking result from the auto tiling system.
<br/><br/>
![Tiling Pattern](Media/BitConfig.png?raw=true "Tiling pattern")
###### edited images from tutsplus.com by Sonny Bone for visual ref
The example image above show the concept of assigning IDs through bitmasks works on four direction.
<br/><br/>
![Tiling Pattern](Media/TileSetSetup.jpg?raw=true "Tiling pattern")  
<br/><br/>
The system would follow a ruleset to generate the Tile Group Sets, each sets IDs
so that no manual data would have to be entered. You can see above the red markings showing a tile
set group and inside, marked with blue, are the tile IDs being cut up.

The patterns that drive these systems can also be driven by a data table to override the default ruleset
that says what type of tile cases should use what combination of four sub tiles to create the desired tile.

Auto tiling patterns and types of behaviour are able to be defined for each tile set.
<br/><br/>
![Tiling Pattern](Media/AutoTilingRock.gif?raw=true "Tiling pattern")
###### gif of auto tiling in engine
<br/><br/>
## Challenges
My limited understanding of bitmasking and binary made the start pretty rough, but after doing some research and practice I picked it up pretty fast.
The general system for bitmasking were pretty straigh forward to implement, the challenges came from building upon the Paper2D editors runtime and editor modules. 
Keeping it fast and simple while allowing the system to still be configured for different types of patterns(and to not break stuff too).

### few code snippets
```cpp
// Bitmask for defining the Tile type to assemble
uint8 index = (NorthWest)|(North << 1)|(NorthEast << 2)|(West << 3)|(East << 4)|(SouthWest << 5)|(South << 6)|(SouthEast << 7);
//...

// itterating through surround tiles checking adject tiles
FPaperTileInfo TargetInk = GetTile(TargetLoc.X, TargetLoc.Y, Layer);
// are we valid and not looking at the same tile?
return (GetTile(TargetLoc.X, TargetLoc.Y, Layer).IsValid() && !TargetInk.AutoTileGroupID == InkID);
//...

// painting editor snippet
    if ((Ink.IsValid() || bBlitEmptyTiles) && (TargetLayer->GetCell(TargetX, TargetY) != Ink))
        {
    		if (!bChangedSomething)
	   		{
				TargetLayer->SetFlags(RF_Transactional);
				TargetLayer->Modify();
				bChangedSomething = true;
			}

			if (FindSelectedComponent()->TileMap->AutoTileEnabled)
			{
				FindSelectedComponent()->AutoTileInkedTile(Ink, TargetX, TargetY, TargetLayer);
			}
			else
			{
				OutDirtyRect += FVector(TargetX, TargetY, LayerCoord);
				TargetLayer->SetCell(TargetX, TargetY, Ink);
			}
//...
```
