---
menu: UMG_Splines
tab: false
parent: Engineering.md
weight: 1
---
# Creating Splines for UMG leveraging the internal drawing API 
![Tiling Pattern](Media/SplineG.gif?raw=true "Tiling pattern")
<br/><br/>
I created UMGSplines leveraging the already existing slate function in FSlateDrawElement, giving 
the option of Cubic or Hermite 2 point splines.
```cpp
FSlateDrawElement::MakeCubicBezierSpline()
```
I created a Base Slate Spline class and a base UMG Spline class. These base classes will hold core functionality 
while I am able to derive from there and build layers of functionality on top or override existing functionality
to easily create Splines with different behavior.
```cpp
// slate
class SSpline;
// uwidget
class USpline;
```
Here is a snippet of the base Slate Spline class and the mandatory SplineData struct. This struct is required for all spline classes, and is passed through to Construct while using SNew or SAssignNew `SNew(USpline, SplineData)` and cannot be set using the postfix `.Member(Value)` notation (named paramater pattern) - as it is not an optional paramater.
```cpp
struct FSplineData
{
public:
	FSplineData()
	{}
	FSplineData(const TArray<FVector2D>& InPoints,
		const FSlateColor& InColor,
		const float& InThickness)
		: Points(InPoints)
		, Color(InColor)
		,Thickness(InThickness)
	{
	}

	TArray<FVector2D> Points;
	FSlateColor Color;
	float Thickness;
};

class UMGSPLINESRUNTIME_API SSpline : public SLeafWidget
{
public:
	// No Slate constructor since this class should be treated as abstract
	virtual FVector2D ComputeDesiredSize(float) const override;
	// Default validation for spline functionality
	virtual const bool IsSplineDataValid() const = 0;

	virtual void CreateSpline(const FPaintArgs& Args, const FGeometry& AllottedGeometry, const FSlateRect& MyCullingRect, FSlateWindowElementList& OutDrawElements, int32 LayerId, const FWidgetStyle& InWidgetStyle, bool bParentEnabled) const;
protected:

	virtual int32 OnPaint(const FPaintArgs& Args, const FGeometry& AllottedGeometry, const FSlateRect& MyCullingRect, FSlateWindowElementList& OutDrawElements, int32 LayerId, const FWidgetStyle& InWidgetStyle, bool bParentEnabled) const override;
protected: 
	// Spline Data
	FSplineData Data;
};
```
Whereas Cubic Bezier Splines takes normally 4 points, start, end and handles a lot of users might want to control there spline with just 
1 control point to simplify things, there is where creating a second point that is evenly distributed would help, building on top of evenly 
placing the control points I am able to allow for evenly distributed bends.
<br/><br/>
![Tiling Pattern](Media/SplineBend.gif?raw=true "Tiling pattern")
###### Spline auto bending and adjusting the postions for a evenly distrabuted curve 
###### Debug Point 1 and 2 show with trigonometry Lines
<br/><br/> 

#### some code snippets
few pieces of basic code that help to drive the behavior of the TriBezierSpline, taking point0 and breaking it down
in to a triangle using the start and end points.
```cpp
void UTriBezierSpline::ConvertPointToTri(const FVector2D& TriPoint, FVector2D& _Point0, FVector2D& _Point1)
{
	FVector2D PointTri(TriPoint);
	PointTri = TriPoint + GetTriOffset();

	_Point0 = FVector2D((Start.X + PointTri.X) * 0.5f, (Start.Y + PointTri.Y) * 0.5f);
	_Point1 = FVector2D((End.X + PointTri.X) * 0.5f, (End.Y + PointTri.Y) * 0.5f);
}
//..
// Generating a even curve to point north
void UTriBezierSpline::BendNorth()
{
	FVector2D NewPoint0 = FMath::Abs(End - Start);

	NewPoint0 = NewPoint0 * 0.5f;
	NewPoint0 = NewPoint0 * -1.f;

	NewPoint0 = NewPoint0 * CurveScale;
	Point0 = NewPoint0;
}
```
This system was expanded on by creating custom containers that then divide them selfs up into sections
and drive the auto-bend configuration of any splines within it.  
<br/><br/>
![Tiling Pattern](Media/AutoBendCanvas.png?raw=true "Tiling pattern")
<br/><br/>
###### Spline auto bending from container driving
###### debug lines show you how the space is split up in to theatre sections

### some code snippets that drive this
```cpp
ESplineDirection USplineMathLibrary::GetSplineDirectionState(const FVector2D Direction)
{
	float SplineDir = GetSplineDirectionAngle(Direction);

	static const TArray<ESplineDirection> SectorDirectionMappings = {
	ESplineDirection::East, ESplineDirection::SouthEast, ESplineDirection::South,
	ESplineDirection::SouthWest, ESplineDirection::West, ESplineDirection::NorthWest,
	ESplineDirection::North, ESplineDirection::NorthEast };

	return SectorDirectionMappings[SplineDir];
}

float USplineMathLibrary::GetSplineDirectionAngle(const FVector2D Direction)
{
	FVector2D _Vec = (Direction - FVector2D(250.f, 250.f)).GetSafeNormal();
	float Angle = FMath::Acos(FVector2D::DotProduct(_Vec, FVector2D(1.f, 0.f)));
	Angle = FMath::RadiansToDegrees(Angle);

	if (_Vec.Y < 0.f)
	{
		Angle = 360.f - Angle;
	}

	const int32 SectorCount = 8;
	const float SectorTheta = 360.0f / SectorCount;
	const float HalfSectorTheta = SectorTheta / 2.0f;

	for (int32 i = 0; i < SectorCount; ++i)
	{
		const float CurrentSectorMin = (SectorTheta * i) - HalfSectorTheta;
		const float CurrentSectorMax = (SectorTheta * i) + HalfSectorTheta;

		if (Angle >= CurrentSectorMin && Angle <= CurrentSectorMax)
		{
			return i;
		}
	}
	return 0;
}
```
### Working around the restrctions of UMGEditor for handles
One of the main challenges I had to face was the restriction of not being able to easily
add a handle based control system in the UMGEditor, there is a system for extending this 
but it is not yet finished. 
###### you can find my PR here finishing exposing the system
###### https://github.com/EpicGames/UnrealEngine/pull/5525
After much testing and iteration, the solution ended up being a function that can be used during
designer time to pass an array of handle targets too the slate widgets, you could then choose to 
delete these handles during the non-designer time to remove them from the widget.
This seems to be the best solution right now until the FDesignerExtension system has been exposed
within the engine, keeping it clean and somewhat easy to decrypt later on.
<br/><br/>
![Tiling Pattern](Media/SplineNode.png?raw=true "Tiling pattern")
<br/><br/>
### Some important things to consider when dealing with robust maintainable code
One thing I had to consider for the future was when I would have access to the FDesignerExtension 
in the engine, so I decided to make the Handle system for the underline slate widgets be an interface.
Having the ability to easily pass the Handle functionality too different spline types and keeping the 
code much easier to read.

### Code Snippet, Slim version of the Handle interface
```cpp
class ISHandle
{
public:
	TArray <TSharedPtr<SWidget>> Handles;
	TSharedPtr<SWidget> MyCanvas;

public:
	virtual void AnchorHandles(FSplineData& Data)
	{
		if (!MyCanvas.IsValid())
		{
			UE_LOG(SplineLog, Warning, TEXT("Warning when attempting to anchor Handles Canvas is invalid"));
			return;
		}

		const auto AllocatedCanvasGeo = MyCanvas->GetCachedGeometry();
		int index = 0;
		for (auto& Handle : Handles)
		{
			if (Handle.IsValid())
			{
				const auto AnchorGeo = Handle->GetCachedGeometry();
				Data.Points[index] = AllocatedCanvasGeo.AbsoluteToLocal(
					AnchorGeo.GetAbsolutePositionAtCoordinates({ 0.f, 0.f }));
			}
			index++;
		}
	}
};
```

### Final thoughts & tackling a robust Spline inheritance system in slate
One thing I really had to consider was the base foundation for SSpline class,
we have our slate macro constructor and are non-macro slate constructor which is mandatory
to create the widget.
These are not to be confused with the class constructor.
For the sake of maintainable code, I decided to wrap the core parameters for a Spline 
in a struct that would be paired with the non-macro constructor, this being the same for 
every derived slate class.
Extra parameter flexibility will come from slate macro constructor allowing for any extra
information to be added on to any classes.
###### Encapsulation, aint nobody got time for that 
#### 1, 2, 3 and easy!
Even the most simple sections of code can lead to system rot without the correct supporting 
framework.
```cpp
FSlateDrawElement::MakeCubicBezierSpline(
		OutDrawElements,
		LayerId,
		AllottedGeometry.ToPaintGeometry(),
		Data.Points[0],
		Data.Points[1],
		Data.Points[2],
		Data.Points[3],
		Data.Thickness,
		ESlateDrawEffect::None,
		Data.Color.GetSpecifiedColor()
```

