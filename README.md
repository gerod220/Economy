<h1 align="center">Economy</h1>
<h3 align="center">Core plugin for creating economic relations between players on cs2 servers</h3>

## Warning
The plugin is unstable and can cause problems.

## Install
1. Install https://github.com/roflmuffin/CounterStrikeSharp
2. Download `Economy.7z`
3. Move the Economy folder to `addons\counterstrikesharp\plugins\`
4. Done

## Usage
On its own, the plugin is nothing of the sort. It does not provide any interface for the player and administration, it is created for developers.

## Features 
- [x] Currency Give, Take, Transfer
- [ ] Items
  - [x] Creation, Purchase, Use, Transfer
  - [x] Events:
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

What follows is a seemingly complex example of how MedKit can be implemented via API.
```csharp
var medKitProduct = new Product(name: "MedKit", description: "Add health for player", money: 1000);
EconomyAPI.RegisterProduct(medKitProduct);
medKitProduct.Purchasing += (userId, product) =>
{
    var myItemMedKit = new Item(product.Name, product.Description, new Dictionary<string, string>() { { "view", $"{ChatColors.Red}MedKit" } });
    myItemMedKit.Using += (userId, item) =>
    {
        var playerController = Utilities.GetPlayerFromUserid(userId);
        playerController.PlayerPawn.Value.Health += 50;

        if (playerController.PlayerPawn.Value.Health > 100) 
            playerController.PlayerPawn.Value.Health = 100;

        return EventResult.Continue;
    };
    var id = EconomyAPI.CreateItem(myItemMedKit);
    EconomyAPI.TakeMoney(userId, product.Money);
    EconomyAPI.GiveItem(userId, id);
    return EventResult.Handled;
};
```
So what's going on here?)
1. We need to create an instance of a class for our product that we will sell to players. Products are mandatory to create, here we only need a purchase event.
2. We register the product to get its Id. Items are created by Id, each item is unique.
3. When the player buys something, Purchasing will be triggered, the purchase has started but not completed, so we can change or cancel the event.
4. Create our instance of the item and subscribe to use it.
5. Simple code, if the player used the item, we change the health, and give the execution of the event to the core. Next, the kernel will delete the item.
6. Create our item already in the kernel. After registering the item we will get its Id, it will be needed to give the item to the player.
7. We are not a charity, so take our money.
8. We give out the item by its Id.
9. Next we announce that we have changed our event. It will return true for the EconomyAPI.PurchaseProduct(userId, itemId) method; but it will not take our money or reissue the item.

I'll leave the rest of the methods and event to self-study, enjoy coding.

## Donate
If you want to communicate and support me to develop this plugin, then write me in Discord: gerod241
