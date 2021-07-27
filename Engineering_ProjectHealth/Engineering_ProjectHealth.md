---
menu: Managing Project Health
tab: false
parent: Engineering.md
weight: 3
---
# Managing Project Data
``I decided to write this up and generate some example content due to a noticed increase of people 
starting projects and developing away without any care for the future health of the project, 
entropy and system rot is an awful thing.``
<br/><br/>
Game projects tend to contain large forms of data, these must be managed and maintained for
the health of the project, leveraging data management tools and implementing checking is a very
robust and strong way to help manage healthy data.

## Primary and secondary Management

Primary and secondary data management comes in many forms, a primary forms tend to impose restrictions
on designers, secondary data management tends to come in play when it comes to checking and verifying
our data is healthy and setup how it should be.

For the examples here, i am going to be demoing the usage of primary and secondary management  for weapon 
data that could be used to drive weapons in a large game.

## Primary Implementation
When it comes to Primary data management restrictions, sometimes it's not as easy as only allowing a certain
type of object/class might not be flexible enough, perhaps some weapons are animated and are skeletal meshes 
or some of them are static meshes, or perhaps some project management has changed, and you have ended up with a 
mix of these, managing our data is important, 1 bad reference can lead to crashing or even worse undefined behaviour.

We can lock some assets out of the detail panel via conditions built in to the data storage class, these are only 
handled at editor time. If the SKM is populated with a valid asset, then the SM slot is locked, and visa versa.
<br/><br/>
![DetailsDisableOther](Media/DetailsDisableOther.gif?raw=true "DetailsDisableOther")

##### code c++
```cpp
// .h
#if WITH_EDITOR
        virtual bool CanEditChange(const FProperty* InProperty) const override;
#endif

// .cpp
#if WITH_EDITOR
bool UWeaponData::CanEditChange(const FProperty* InProperty) const
{
    // lets check if other logic prevents CanEditChange
    const bool ParentVal = Super::CanEditChange(InProperty);

    if (InProperty->GetFName() == GET_MEMBER_NAME_CHECKED(UWeaponData, SKM))
    {
        return ParentVal && !SM;
    }

    if (InProperty->GetFName() == GET_MEMBER_NAME_CHECKED(UWeaponData, SM))
    {
        return ParentVal && !SKM;
    }

    return ParentVal;
}
#endif
```
This is a very simple example, this can be leveraged to do more advanced things.
An example could be having a drop down type like a enum where you can set a property of a 
weapon and then have this logic go through and disable / enable things that are relevant to 
your defined property.
###### note
`There is also a metadata tag called "Name_EditConditionHides" that has the ability to hide
FProperty when there CanEditChange returns false, however currently there appears to be no 
way to apply this meta tag via UProperty macro's.`

## Secondary Implementation

Sometimes it is not plausible to impose these sorts of restrictions, or perhaps this sort of data 
health management is being applied midway during development, so some things could possibly slip 
through the crack. We need a method to validate our data, this is where it's important for most projects
to leverage the Data Validation Plugin that is included with the editor, this allows us to add editor
only data validation, this is supported by all UObject derived classes by default.

Data validation allows our WeaponData class to override IsDataValid function, that returns 
EDataValidationResult enum, this enum can be Invalid, Valid and NotValidated.

Our data validation function is set up to check if there is a valid mesh and if there is a valid material,
either of these being invalid will cause the data validation to fail.
<br/><br/>
![DVOutputFail](Media/DVOutputFail.png?raw=true "DVOutputFail")

##### code c++
```cpp
// .h
#if WITH_EDITOR
		EDataValidationResult IsDataValid(TArray<FText>& ValidationErrors) override;
#endif

// .cpp
#if WITH_EDITOR
EDataValidationResult UWeaponData::IsDataValid(TArray<FText>& ValidationErrors)
{
    Super::IsDataValid(ValidationErrors);
  
    if (!SKM & !SM)
    {
        // here is where we would print more information to the game project defined log
        return EDataValidationResult::Invalid;
    }

    if (!Material)
    {
        // here is where we would print more information to the game project defined log
        return EDataValidationResult::Invalid;
    }

    return EDataValidationResult::Valid;
}
#endif 
```
You would choose to add more logging in to the functions to give some more accurate feedback on
why some assets have failed validation or why they can not be validated.
Data validation can easily be done on a select asset, folder or even the whole project.

## Safety with changing base data types

While all these are small but very powerful tools to help nurture your data through its lifetime,
sometimes it's just not good enough to catch all use cases, or tracking down undefined behaviour can 
be a nightmare, perhaps a junior level designer has used some data that's deprecated or is performing 
some form of operating on the data that is implicitly using something it should not be, tracking 
things like this can be hard, however we can leverage UMACRO Properties to help manage some of this,
understanding deprecating data is important, we can't always hot swap or remove class members.

<br/><br/>
![DeprecatedWarning](Media/DeprecatedWarning.png?raw=true "DeprecatedWarning")

##### code c++
```cpp
// .h         
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Visual", meta = (DeprecatedProperty, DeprecationMessage="Texture is no longer used, please use a material"))
     UTexture* Texture;
```
Accessing this data member now causes Deprecated warnings in the BP compiler.
Being able to deprecate data appropriately is very important, project health can be such a fragile thing,
displaying proper verbosity for problems is important, as the once great engineer Allar`(https://allarsblog.com/)`
said "Treat Warnings as Errors, Treat Errors as Sins" this is a strong way to avoid cascading problems.