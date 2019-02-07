# Mikroprozessorpraktikum
## Konstantin Bork & Kean Seng Liew, Gruppe A, HWP8
### 07-02 Stop Mode
#### A07-02.1
> Die Aufgabenstellung ist identisch zu Aufgabe A07-01.1. Allerdings soll hier der Low Power Stop Mode anstelle des Sleep
Mode eingesetzt werden. Bewerten Sie die gemachten Beobachtungen hinsichtlich der Einsetzbarkeit und der Stromersparnis. 

#### main.c

    #include "main.h"
    #include "aufgabe.h"
    
    int main(void)
    {
        // Initialisierung des Systems und des Clocksystems
        SystemInit();
    
        // Initialisiere die grüne LED
        init_leds();
    
        // Initialisiere beide Tasten
        init_taste_1();
        init_taste_2_irq();
    
    	init_usart_2_irq();
    
    	init_nvic();
    
        usart_2_print("\r\nSystem Neustart\r\n");
    
        while(1)
    	{
        	wait_mSek(3000);
        	GR_LED_Toggle;
    
        	int taste_1_gedruckt = taste_1_gedrueckt();
    
        	if(taste_1_gedruckt == 0)
        	{
        		GR_LED_ON;
        		usart_2_print("\r\nStop Mode Start\r\n");
    
        		// Stop Mode aktivieren
        		PWR_FlashPowerDownCmd(ENABLE);
        		PWR_EnterSTOPMode(PWR_Regulator_LowPower, PWR_STOPEntry_WFI);
        		PWR_FlashPowerDownCmd(DISABLE);
    
        		// Initialisiere wieder das System
        		SystemInit();
    
        		usart_2_print("\r\nStop Mode Ende\r\n");
        	}
       	}
    }

Die anderen relevanten Dateien wurden nicht verändert.  

Unser Ausgangsverbrauch beträgt 3,9V * 185mA = 721,5mW, im Stop Mode haben wir 3,9V * 140mA = 546mW gemessen.
Das bedeutet eine Ersparnis von 45mA bzw. 175,5 mW, also mehr als im Sleep Mode. Die Verwendung des Stop Mode
bedeutet mehr Arbeit, da man beim Austritt aus dem Stop Mode an die Initialisierung des Clock-Systems
denken muss. Allerdings ist dieser Aufwand noch immer überschaubar und auch wert, wenn man mehr Strom
sparen muss.