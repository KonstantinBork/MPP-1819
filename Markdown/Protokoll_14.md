# Mikroprozessorpraktikum
## Konstantin Bork & Kean Seng Liew, Gruppe A, HWP8
### 07-03
#### A07-03.1
> Im nächsten Schritt soll die Nutzung der RTC gemeinsam mit dem Standby Mode untersucht werden. Dazu soll ein Programm
> mit folgenden Eigenschaften entwickelt werden.
> - Die RTC soll in der main() Funktion noch vor der Endlosschleife initialisiert (RTC WakeUp zyklisch alle 30 Sekunden) werden.
> - In einer Endlosschleife soll die grüne LED im Sekundentakt basierend auf einer waituSek() Funktion blinken.
> - In der gleichen Endlosschleife soll die Taste1 im Polling abgefragt werden. Wurde die Taste gedrückt sollen folgende Teilschritte absolviert werden
>  - die grüne LED soll eingeschaltet werden
>  - die Wake Up Funktionalität der eines RTC Wake Up aktiviert werden
>  - nach einer Wartezeit von 2 Sekunden soll in den Standby-Mode gewechselt werden
>
> Testen Sie das Programm und Beobachten Sie den Stromverbrauch.

#### main.c

    #include "main.h"
    #include "aufgabe.h"
    
    int main(void)
    {
        // Initialisierung des Systems und des Clocksystems
        SystemInit();
    
        // SysTick initialisieren
        // jede ms erfolgt dann der Aufruf
        // des Handlers fuer den Interrupt SysTick_IRQn
        InitSysTick();
    
        // Start der RTC  falls diese noch
        // nicht initialisiert war wird
        // die RTC mit der LSE-Taktquelle aktiviert
        start_RTC();
    
        // Initialisiere die grüne LED
        init_leds();
    
        // Initialisiere beide Tasten
        init_taste_1();
    
        usart_2_print("\r\nSystem Neustart\r\n");
    
        // Initialisiere die RTC wie in der Aufgabe beschrieben
        init_rtc_wakeup_30sec();
    
        while(1)
    	{
        	wait_mSek(1000);
        	GR_LED_Toggle;
    
        	int taste_1_gedruckt = taste_1_gedrueckt();
    
        	if(taste_1_gedruckt == 0)
        	{
        		GR_LED_ON; // Schalte die LED ein
        		activate_wakeup_event(); // aktiviere das Wakeup-Event
    
        		usart_2_print("\r\nStandby Mode Start\r\n");
        		wait_mSek(2000); // Warte 2 Sekunden
        		PWR_EnterSTANDBYMode(); // Wechsle in den Standby Mode
        	}
        }
    }

#### Auszug aufgabe.h

    // Aufgabe A07-03.1
    void init_rtc_wakeup_30sec();
    void activate_wakeup_event();

#### Auszug aufgabe.c

    void init_rtc_wakeup_30sec()
    {
    	// Strukt anlegen
    	EXTI_InitTypeDef EXTI_InitStructure;
    	NVIC_InitTypeDef NVIC_InitStructure;
    
    	// EXTI-Line Konfiguration für WakeUp
    	EXTI_ClearITPendingBit(EXTI_Line22);
    	EXTI_InitStructure.EXTI_Line = EXTI_Line22;
    	EXTI_InitStructure.EXTI_Mode = EXTI_Mode_Interrupt;
    	EXTI_InitStructure.EXTI_Trigger = EXTI_Trigger_Rising;
    	EXTI_InitStructure.EXTI_LineCmd = ENABLE;
    	EXTI_Init(&EXTI_InitStructure);
    
    	// NIVC Konfiguration für WakeUp
    	NVIC_InitStructure.NVIC_IRQChannel = RTC_WKUP_IRQn;
    	NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 0;
    	NVIC_InitStructure.NVIC_IRQChannelSubPriority = 0;
    	NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
    	NVIC_Init(&NVIC_InitStructure);
    
    	RTC_WakeUpCmd(DISABLE);
    
    	// Konfiguration der Clock
    	// RTC_WakeUpClock_RTCCLK_Div16;   (976,562usek...32sek)Zeitbasis: 488,281us
    	RTC_WakeUpClockConfig(RTC_WakeUpClock_RTCCLK_Div16);
    
    	// RTC_WakeUpCouter mit einem Wert zwischen 0x0000...0xFFFF setzen
    	// Wert des WakeUpCounter aus Zeitintervall/Zeitbasis berechnen
    	// WakeUpCounterwert für intervall von 30s bei RTC_WakeUpClock_RTCCLK_Div16
    	// ergibt sich aus 30s/488,281us = 61440
    	RTC_SetWakeUpCounter(61440);
    }
    
    void activate_wakeup_event()
    {
    	// Disable RTC wakeup interrupt
    	RTC_ITConfig(RTC_IT_WUT,DISABLE);
    
    	// Clear PWR Wakeup WUF Flag
    	PWR_ClearFlag(PWR_CSR_WUF);
    	PWR_WakeUpPinCmd(ENABLE);
    
    	// Clear RTC Wakeup WUTF Flag
    	RTC_ClearITPendingBit(RTC_IT_WUT);
    	RTC_ClearFlag(RTC_FLAG_WUTF);
    
    	// Freigeben
    	RTC_ITConfig(RTC_IT_WUT, ENABLE);   // Bit 14
    	RTC_AlarmCmd(RTC_CR_WUTE, ENABLE);  // Bit 10
    
    	RTC_WakeUpCmd(ENABLE);
    }

Der Ausgangsverbrauch beträgt 3,9V * 185mA = 721,5mW, im Stop Mode haben wir 3,9V * 135mA = 526,5mW gemessen.
Die Ersparnis ist deutlich geringer als erwartet, besonders im Gegensatz zum Stop Mode ist der Aufwand
den zusätzlichen Aufwand laut unseren Beobachtungen nicht wert eingesetzt zu werden. Möglicherweise
liegt ein Fehler auf unserer Seite vor, besonders unsere Messung kann an der falschen Stelle erfolgt
sein.