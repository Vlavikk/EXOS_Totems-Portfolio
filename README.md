# Totems
Custom effects with passive and active effects. Each totem has attributes:
- Effect in the player's hand (passive or active)
- Effect after resurrection

## All Totems
![Image](https://github.com/user-attachments/assets/1f67b54d-21e7-4230-9bc4-b8ae42b41673)

### Totem example
Traveler Totem:

- When receiving damage, the player half-flies a short-term acceleration
- When destroyed, it teleports the player to the nearest safe place

https://github.com/user-attachments/assets/7a818280-208d-4af5-9557-358050d1f092

Illumination Totem:

- When a player right-clicks, it highlights all the players in the radius
- When broken, it gives the effect of an ordinary totem

https://github.com/user-attachments/assets/fce8d353-8618-44fb-8dab-1af573a505a0

## Admin
Give Totems:
<p align="start">
 <img width="200" src="https://github.com/user-attachments/assets/0f08b547-3e4c-4603-81e0-e401db570d81" alt="snake"/>
</p>

- Each totem has a CustomModelData
- Item can be added to any loot table

It is easy to create a new totem through an abstract class
### Simple example
Fire Totem:
```java
public class FireTotem extends AbstractTotem {
    @Override
    public void onResurrectTotem(EntityResurrectEvent e) {
        Player player = (Player) e.getEntity();
        player.addPotionEffect(new PotionEffect(PotionEffectType.FIRE_RESISTANCE, 800, 0)); //Дефолтный эффект ломания тотема
    }
    @Override
    public void playerEffectOnEquip(Player player) {
        addDefaultEffectTotem(player,new PotionEffect(PotionEffectType.FIRE_RESISTANCE, -1, 0));
    }
    @Override
    public void playerEffectOnRemove(Player player) {
        removeDefaultEffectTotem(player,PotionEffectType.FIRE_RESISTANCE);
    }
}
```
Absorption Totem:
```java
public class AbsorptionTotem extends AbstractTotem {
    private final HashSet<Player> playersWithTotem = new HashSet<>();

    public AbsorptionTotem() {
        setDamageTask();
    }

    @Override
    public void playerEffectOnEquip(Player player) {
        addDefaultEffectTotem(player,new PotionEffect(PotionEffectType.HASTE, -1, 1));
        playersWithTotem.add(player);
    }
    @Override
    public void playerEffectOnRemove(Player player) {
        removeDefaultEffectTotem(player,PotionEffectType.HASTE);
        playersWithTotem.remove(player);
    }
    @Override
    public void onResurrectTotem(EntityResurrectEvent e) {
        Player player = (Player) e.getEntity();
        player.setHealth(0);

        new ParticleBuilder(Particle.SOUL)
                .location(player.getLocation())
                .offset(0.2,1,0.2)
                .count(25)
                .spawn();
    }
    private void setDamageTask(){
        new BukkitRunnable() {
            @Override
            public void run() {
                TotemManager totemManager = EXOS_Totems.getTotemManager();
                for (Player player : playersWithTotem){
                    if (totemManager.getAbstractTotemByItemStack(player.getInventory().getItemInOffHand()) instanceof AbsorptionTotem){
                        player.damage(1);
                    }else playersWithTotem.remove(player);
                }
            }
        }.runTaskTimer(EXOS_Totems.getInstance(),80,80);
    }
}
```

