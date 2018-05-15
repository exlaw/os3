
/*++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
                               tty.c
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
                                                   
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*/

#include "type.h"
#include "const.h"
#include "protect.h"
#include "string.h"
#include "proc.h"
#include "tty.h"
#include "console.h"
#include "global.h"
#include "keyboard.h"
#include "proto.h"

#define TTY_FIRST	(tty_table)
#define TTY_END		(tty_table + NR_CONSOLES)
PRIVATE void init_tty(TTY* p_tty);
PRIVATE void tty_do_read(TTY* p_tty);
PRIVATE void tty_do_write(TTY* p_tty);
PRIVATE void put_key(TTY* p_tty, u32 key);
PRIVATE void search(TTY* p_tty);
PRIVATE void remove(TTY* p_tty);
extern int to_clear;
unsigned int tabs[100];
int tabs_len;
int color= 0 ;
int isESC=0;
int isEnter=0;
u32 input_search[100];
int search_len=0;
/*======================================================================*
                           task_tty
 *======================================================================*/
PUBLIC void task_tty()
{
    isESC=0;
    isEnter=0;
    search_len=0;
        tabs_len = 0;
	TTY*	p_tty;
	init_keyboard();
	for (p_tty=TTY_FIRST;p_tty<TTY_END;p_tty++) {
		init_tty(p_tty);
	}
	select_console(0);
	while (1) {
		for (p_tty=TTY_FIRST;p_tty<TTY_END;p_tty++) {
			//if (to_clear == 1)
                     //   {
                     //      clear_screen(p_tty->p_console);
                      //     to_clear = 0;
                      //  }
                        if (to_clear == 1&& is_current_console(p_tty->p_console))
                        {
                        while(p_tty->p_console->cursor > 0)
		        out_char(p_tty->p_console,'\b');
                        to_clear = 0;
			}
			tty_do_read(p_tty);
			tty_do_write(p_tty);
		}
	}
}

/*======================================================================*
			   init_tty
 *======================================================================*/
PRIVATE void init_tty(TTY* p_tty)
{   

	p_tty->inbuf_count = 0;
	p_tty->p_inbuf_head = p_tty->p_inbuf_tail = p_tty->in_buf;

	init_screen(p_tty);
}

/*======================================================================*
				in_process
 *======================================================================*/
PUBLIC void in_process(TTY* p_tty, u32 key)
{
        char output[2] = {'\0', '\0'};
	int i =0;
        unsigned int  k = 0 ;
		//查找模式下按下Enter查找并屏蔽所有输入
		
          
              
        if (!(key & FLAG_EXT)&&!(isESC==1&&isEnter==1)) {
        	
        		
                if(isESC==1&&isEnter==0){
                       input_search[search_len]=key;
                       search_len++;
                }
		put_key(p_tty, key);
        }
        else {
                int raw_code = key & MASK_RAW;
                switch(raw_code) {
                case ENTER:
                    if(isESC==1&&isEnter==1)
                          break;
				    if(isESC==1)
						isEnter=!isEnter;
					if(isEnter==1&&isESC==1){
						search(p_tty);
						break;
					}
			        put_key(p_tty, '\n');
			         break;
			    case ESC:
				     isESC=!isESC;
					 if(isESC==0){
					 	
						 remove(p_tty);
						 break;
					 }
					 if(isESC==1){
					    color=1;
					    disable_irq(CLOCK_IRQ);  
					 }   
			       	
					 break;
                case BACKSPACE:
                if(isESC==1&&isEnter==1)
                          break;
			put_key(p_tty, '\b');
			break;
                case UP:
                if(isESC==1&&isEnter==1)
                          break;
                        if ((key & FLAG_SHIFT_L == FLAG_SHIFT_L) || (key & FLAG_SHIFT_R)) {
				scroll_screen(p_tty->p_console, SCR_DN);
                        }
			break;
		case DOWN:
		if(isESC==1&&isEnter==1)
                          break;
			if ((key & FLAG_SHIFT_L) || (key & FLAG_SHIFT_R)) {
				scroll_screen(p_tty->p_console, SCR_UP);
			}
			break;
                case TAB:
                   if(isESC==1&&isEnter==1)
                          break;
                     for(i = 0;i < 4; ++i)
                       put_key(p_tty,' ');
                  //   if (tabs_len > 2)
                    //  tabs[tabs_len++] = p_tty->p_console->cursor;
                     //else 
                      tabs[tabs_len++]= p_tty->p_console->cursor;
                     break;
		case F1:
		case F2:
		case F3:
		case F4:
                   
		case F5:
                     
		case F6:
		case F7:
		case F8:
		case F9:
		case F10:
		case F11:
		case F12:
			/* Alt + F1~F12 */
			if ((key & FLAG_ALT_L) || (key & FLAG_ALT_R)) {
				select_console(raw_code - F1);
			}
			break;
                default:
                        break;
                }
        }
}

/*======================================================================*
			      put_key
*======================================================================*/
PRIVATE void put_key(TTY* p_tty, u32 key)
{
	if (p_tty->inbuf_count < TTY_IN_BYTES) {
		*(p_tty->p_inbuf_head) = key;
		p_tty->p_inbuf_head++;
		if (p_tty->p_inbuf_head == p_tty->in_buf + TTY_IN_BYTES) {
			p_tty->p_inbuf_head = p_tty->in_buf;
		}
		p_tty->inbuf_count++;
	}
}
PRIVATE void search(TTY* p_tty){
	find_search(p_tty->p_console);
}

PRIVATE void remove(TTY* p_tty){
	isESC=0;
	isEnter=0;
	color=0;
	
    remove_search(p_tty->p_console);
    search_len=0;
    enable_irq(CLOCK_IRQ);     

}
/*======================================================================*
			      tty_do_read
 *======================================================================*/
PRIVATE void tty_do_read(TTY* p_tty)
{
	if (is_current_console(p_tty->p_console)) {
		keyboard_read(p_tty);
	}
}


/*======================================================================*
			      tty_do_write
 *======================================================================*/
PRIVATE void tty_do_write(TTY* p_tty)
{
	if (p_tty->inbuf_count) {
		char ch = *(p_tty->p_inbuf_tail);
		p_tty->p_inbuf_tail++;
		if (p_tty->p_inbuf_tail == p_tty->in_buf + TTY_IN_BYTES) {
			p_tty->p_inbuf_tail = p_tty->in_buf;
		}
		p_tty->inbuf_count--;

		out_char(p_tty->p_console, ch);
	}
}


