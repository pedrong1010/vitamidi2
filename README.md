# vitamidicontroller
#include <psp2/kernel/processmgr.h>
#include <psp2/ctrl.h>
#include <psp2/touch.h>
#include <psp2/display.h>
#include <stdio.h>
#include <string.h>

#define MIDI_CHANNEL 0x90 // Canal 1 para Note On
#define MIDI_CC 0xB0      // Control Change
#define MIDI_PB 0xE0      // Pitch Bend

void send_midi(unsigned char status, unsigned char data1, unsigned char data2) {
    unsigned char midi_msg[3] = {status, data1, data2};
    // Função para enviar via USB (depende da implementação específica)
    usb_send_midi(midi_msg, 3);
}

int main() {
    SceCtrlData pad;
    SceTouchData touch;
    sceCtrlSetSamplingMode(SCE_CTRL_MODE_ANALOG);
    
    while (1) {
        sceCtrlPeekBufferPositive(0, &pad, 1);
        sceTouchPeek(SCE_TOUCH_PORT_FRONT, &touch, 1);
        
        // Mapeamento de botões para MIDI Note ON/OFF
        if (pad.buttons & SCE_CTRL_CROSS) send_midi(MIDI_CHANNEL, 60, 127); // Nota C4
        if (pad.buttons & SCE_CTRL_CIRCLE) send_midi(MIDI_CHANNEL, 62, 127); // Nota D4
        if (pad.buttons & SCE_CTRL_SQUARE) send_midi(MIDI_CHANNEL, 64, 127); // Nota E4
        if (pad.buttons & SCE_CTRL_TRIANGLE) send_midi(MIDI_CHANNEL, 65, 127); // Nota F4
        
        // Pitch Bend pelos analógicos
        int pitch_bend = (pad.lx - 128) * 64;
        send_midi(MIDI_PB, pitch_bend & 0x7F, (pitch_bend >> 7) & 0x7F);
        
        // Controle de faders pela tela de toque
        if (touch.reportNum > 0) {
            int touch_x = touch.report[0].x / 16;
            int touch_y = touch.report[0].y / 16;
            send_midi(MIDI_CC, 10, touch_x); // Controle X
            send_midi(MIDI_CC, 11, touch_y); // Controle Y
        }
        
        sceKernelDelayThread(10000); // Pequeno delay para evitar flooding
    }
    
    sceKernelExitProcess(0);
    return 0;
}
