# PV_PWM_HEATERv2
photovoltaic MPPT controller for heat buffer
## MPPT PWM 400V DC 10A heater controller board

![schema](/Photo/assy2.jpg)
Second version of controller

###  ◾ About this project
Project was developed to maximize energy transfer from 3,5kW solar array into heaters installed in 1000l heat buffer.
You can find similar systems already in market, ready to purchase but no one had power that I needed.
Below you can find schematic of the idea how to store small, medium and big amounts of heat generated by DC current from solar array.

###  ◾ Overall concept - how to store 50kWh of energy in form of heat and why
Some time ago I built heat buffer to collect and store heat from available heat sources. Basic concept was to store heat from buring coal or wood (furnace is more efficient working at full power and produces less air pollutants). Electric heaters were installed "just in case" of further development. Heat is used for heating whole house and tap water as well.

Using PV array to produce heat sounds as not particularly bright idea: during wintertime there is no enough sun weather to produce reasonable amount of energy (from PV array) to heat whole building (I live at 50* latitude in Europe).
But summertime (autumn and spring usually as well) makes big difference. 5 people cannot use all heat produced by this system.
So during summertime I use generated heat to fill small pool (500liters) for kids - already with warm water, no waiting to warm-up, fresh and hot water in pool every day.
Also heat buffer is large enough that hot water stored from two sunny days lasts for next two days even if they are cloudy.
schematic of that system below

![schema](/Photo/system_sch_diagram.png)
###  ◾ Why two heaters
1. To make use of layered heat tank. If particular day is not excessively sunny then controller heats only upper part of the tank (200l of water).
2. if there is clear sky, long sunny day AND there is already hot water in upper part of the tank then controller stops using upper heater and drives only the lower heater
3. one heater (in form of serially connected two 230V 2kW heaters) can't put anough load to solar array. you need a bit more load so: in full sunshine upper heater works with 100% and lower just about 20% of possible load - only then you can mantain maximum power point (in my case somewhere ~360V DC).

###  ◾ MPPT Controller theory of operation ('MPPT' tracking)
Generally speaking you can't connect heaters directly as a load into solar array. Or at least you can, but you will loose lot of energy.
You can find lot of explanations why this is so.
So you have to track maximum power point of your solar array, controllers that can do that are called MPPT and they are usually very expensive (although there is reason for that). I thought that there is no sense of converting DC energy into AC energy to power simple electric heater. So after small research ([see sample project from youtube]( https://www.youtube.com/watch?v=ciyA7mSo7IY)) i decided to built my own controller board.

**Maximum-Power-Point-Tracking**
To be clear: controller doesn't follow power point of solar array, it follows voltage. I added temperature correction (about 10V) that depends on temperature of NTC.
In my case this NTC measures ambient temperature. To properly follow MPP of PV array you should measure temperature of PV array not ambient.
in my opinion this solution is good enough but not perfect.
Perfect solution should be based on microcontroller that calculates actual power not just voltage.

**General idea is to use RC circuit to oscillate around MPP of solar array.**
![schema](/Photo/ideaofoperation.png)
Idea of RC oscillator (op-amp based) is pretty simple. The issue was that array of that size produces 360VDC (disconnected even 440VDC) and almost 10A continuosly: RC oscillators usually have much less power.

So some simulations and futher research was needed.
Below you can find LTSpice simulation results for partially cloudy day (50% of PWM), just for simulation I set MPP at 330V.
![schema](/Photo/LTSPICE.png)

green curve shows oscillations around MPP at 360VDC (actually it is set by potentiometer on the board).
Hysteresis at 360V MPP was set to 5V.
transistor connects the load (heater) and main capacitor acts as short-term energy source.
Capacitor must withstand current of 10A so special low ESR capacitor is needed (4.7uF 630V DC 6mR from EPCOS)
transistor must be good as well: any inductance produces voltage spikes that doubles overall voltage between drain and source. And even driving mosfet with 100% PWM makes thing hard because 10A current produces conductivity loses.

Picture below shows Drain-Source voltage of SiC transistor (Infineon's IMZ120R030M1HXKSA1).
![schema](/Photo/DS2_QuickPrint43.png)

###  ◾ Actual idea of operation for this controller
All in all I don't want to make here detailed description about part selection. Critical components are big cap and two power transistors. I tested Infineons SiC parts IMZ120R030M1HXKSA1 and IMZ120R045M1XKSA1 toogether with EPCOS B32674D6475K000 4u7 and B32774D8126K000 12uF (big cap gives acoustic noise due to low frequency of operation BUT low frequency generates less commuting loses).

#### voltage set point

high priority heater (upper one) has circuit with mean voltage operation set point around 350VDC.

the lower priority heater has voltage set-point 10V higher ~360V DC.

In case of partial sun/low energy producing PV array (due to low sun early in the morning, or due to cluds) controller uses only high pririty heater (upper in the heat buffer) and operates around 350V DC.

When PV array produces more than 2,5kW of power one heater (~52ohm) cann't operate at MPP for PV array - voltage raises even to 380VDC. No problem for the heater but operating at that high voltage is out of MPP for my arrray.
So there comes into action second heater with second (10V higher) voltage set-point:
upper heater is driven 100% PWM, and second (physically lower in heat buffer) heater is driven with PWM that enables PV to operate at 360V DC.
PV array can then operate at voltage point close to MPP.

if there is already hot water in the tank AND sunny weather continues the controller stops driving both heaters and gives signal thru optocouplers: you can use that to control classic grid inverter to generate electricity and return that to the grid. But heaters have priority in that case and energy can be sent to grid only when there is no need for generatin more heat.

###  ◾ Safety of operation
there is no strict SIL biult-in in that controller. I would recommend some double security to preven fire (due to overheating of electronics itself) or to prefent pressure biuldup in heater tank (in case of NTC failure).
My system has only one level of safety for every critical component.
1. thermostat for transistor heatsink - over 100C it disconnects voltage sensing - controller stops operation
2. 2x NTC at heat buffer - to preven overheating/boiling water. But if NTC fails due to internal malfunction OR due to lack of sufficient thermal connection to tank OR if there is short on power transistor *there is no other security measure to preven overheating
3. to prevent pressure build-up (in case of NTC failure or transistor malfunction ) that tank itself has an overflow port - it allows to escape if there is any excessive prese, no matter from water or from steam
4. to prevent any burn due to very hot tap vater I installed heat regulating valve at buffer output, it is set to 55C

### Similar solution
https://www.ebay.de/itm/Hezspirale-Heizelement-Heizstab-6000-W-3x2000W-230-V-Abdeckung-/221984855525
here similar solution, but with only one channel , smaller voltage (for smaller PV array)

###  ◾ Main components
Sorry for choosen language but this is the exact part number that I used.
heater
https://www.egniazdka.pl/grzalka-do-bojlera-z-kolpakiem-eliko-grbk-3x2000w-u-64-p-1829.html
low ESR capacitor
https://pl.farnell.com/epcos/b32674d6475k000/cap-4-7-f-630v-10-pp/dp/2728560
SiC transistor
https://pl.farnell.com/infineon/imz120r030m1hxksa1/mosfet-n-ch-1-2kv-56a-175deg-c/dp/3223684
DCDC converter
https://pl.farnell.com/recom-power/rac10-12sk-277/power-supply-ac-dc-12v-0-84a/dp/2822840

###  ◾ BOM cost
I had to build two versions + during testing I burned one transistor (due to DC arc) so this was not completely cheap project.
but properly assembled ona piece of that board should cost you about:
* PCB JLCPCB $20
* main transistors, capacitor, DCDC converter other major components about $100

