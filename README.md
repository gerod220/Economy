<h1 align="center">Economy</h1>
<h3 align="center">Core plugin for creating economic relations between players on cs2 servers</h3>

## Install
1. Install https://github.com/roflmuffin/CounterStrikeSharp
2. Download `Economy.zip`
3. Move the Economy folder to `addons\counterstrikesharp\plugins\`
4. Done

## Usage
On its own, the plugin is nothing of the sort. It does not provide any interface for the player and administration, it is created for developers.

## Features 
- [x] Currency Give, Take, Transfer
- [ ] Items
  - [x] Creation, Purchase, Use, Transfer
  - [ ] Events:
    - [x] Purchasing, Using, Transfer
    - [ ] Created, Destroyed
  - [x] Attributes
- [x] Hook Events (Modules can modify events that are sent by the Economy plugin)
- [x] CacheManager
- [ ] SQLite/MySQL
- [ ] Config (You can change the max amount, but it doesn't make sense yet xD. Mainly serves to switch debugging)
- [ ] ModuleAPI (system so that the modules can communicate with each other. For example, if a store module allows you to create a category, another module will be able to use the function to create its own category.)
- [ ] etc..

## API
Best of all, this plugin is built so that its functionality is modules.
A module is a plugin from another developer that uses EconomyAPI to create its own economic relationship between players.

1. Create CSS plugin
2. Add EconomyAPI.dll your project.
3. Use Economy.EconomyAPI

```cs
public override async void Load(bool hotReload)
{
    await EconomyAPI.RegisterModule(this);
}
```
As you notice, this is an asynchronous method, which means that the server will not hang if the kernel is not there, and at the same time will not allow an error to be generated because the API functionality will not work.

# Events
since version 0.2-alpha events have been rewritten to attributes

- Method Signatures:

ProductPurchasing(iUserId, GuidProductId)
```csharp
[EconomyEvent(EventId = EconomyEvents.ProductPurchasing]
public static EventResult OnProductPurchasing(int userId, Guid productId)
{
    return EventResult.Continue;
}
```
ProductPurchased(iUserId, GuidProductId)
```csharp
[EconomyEvent(EventId = EconomyEvents.ProductPurchased]
public static void OnProductPurchased(int userId, Guid productId)
{

}
```
ItemUsing(iUserId, GuidItemId)
```csharp
[EconomyEvent(EventId = EconomyEvents.ItemUsing]
public static EventResult OnItemUsing(int userId, Guid itemId)
{
    return EventResult.Continue;
}
```
ItemUsed(iUserId, GuidItemId)
```csharp
[EconomyEvent(EventId = EconomyEvents.ItemUsed]
public static void OnItemUsed(int userId, Guid itemId)
{

}
```
MoneyTransfer(dMoney, iFormerUserId, iNewUserId)
```csharp
[EconomyEvent(EventId = EconomyEvents.MoneyTransfer]
public static void OnItemUsed(decimal amount, int formerUserId, int newUserId)
{

}
```
ItemTransfer(GuidItemId, iFormerUserId, iNewUserId)
```csharp
[EconomyEvent(EventId = EconomyEvents.ItemTransfer]
public static void OnItemUsed(Guid itemId, int formerUserId, int newUserId)
{

}
```

- Attribute parameters
In order to have less code in the handler, the basics have been put into parameters, for example product/item Id should be specified in an attribute.
```csharp
[EconomyEvent(EventId = EconomyEvents.ProductPurchasing, Id = "83dba2c3-1f3e-4d8e-9bb4-0070358cf68c"]
public static EventResult OnProductPurchasing(int userId, Guid productId) // productId = Id
{
    return EventResult.Continue;
}
```

As you have noticed, items support custom attributes, you can specify at which item attributes the handler will work
```csharp
[EconomyEvent(EventId = EconomyEvents.ItemUsing, HasAttributes = new string[] { "productId" } ]
public static EventResult OnItemUsing(int userId, Guid itemId)
{
    var item = EconomyAPI.GetItemById(Guid.Parse(itemId))
    var productId = item.Attributes["productId"]; // The item will always have the attribute, otherwise the handler will not be called.
    return EventResult.Continue;
}
```

What follows is a seemingly complex example of how MedKit can be implemented via API.
```csharp
private const string MedKitId = "4c8ca223-8b28-41b4-8c7d-66b92b7566c8";

[EconomyEvent(EventId = EconomyEvents.ProductPurchasing, Id = MedKitId)]
public static EventResult OnMedKitPurchasing(int userId, Guid productId)
{
    var product = EconomyAPI.GetProductById(productId);
    var myItemMedKit = new Item(product!.Name, product.Description, new Dictionary<string, string>()
    {
        { "view", $"{ChatColors.Red}MedKit" },
        { "productId", $"{product.Id}" }
    });

    if(EconomyAPI.TakeMoney(userId, product.Money))
    {
        var id = EconomyAPI.CreateItem(myItemMedKit);
        EconomyAPI.GiveItem(userId, id);
    }
    return EventResult.Handled;
}

[EconomyEvent(EventId = EconomyEvents.ItemUsing, HasAttributes = new string[] { "productId" })]
public static EventResult OnItemUsing(int userId, Guid itemId)
{
    var item = EconomyAPI.GetItemById(itemId);
    if (item.Attributes["productId"] == MedKitId.ToString())
    {
        var playerController = Utilities.GetPlayerFromUserid(userId);
        playerController.PlayerPawn.Value.Health += 50;
        if (playerController.PlayerPawn.Value.Health > 100)
            playerController.PlayerPawn.Value.Health = 100;
    }
    return EventResult.Continue;
}
```

I'll leave the rest of the methods and event to self-study, enjoy coding.

## Donate
If you want to communicate and support me to develop this plugin, then write me in Discord: gerod241
