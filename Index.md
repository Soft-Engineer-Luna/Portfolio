---
menu: Index 
tab: false
weight: 0
---

 [![Foo](Engineering_UMG_Splines/Media/SplineG.gif?raw=true)](Engineering_UMG_Splines/Engineering_UMG_Splines.html)



 ``` cpp

	if (!SplineThickness.IdenticalTo(_SplineThickness))
	{
		SplineThickness.Set(_SplineThickness);
		Invalidate(EInvalidateWidget::PaintAndVolatility);
	}

 ```