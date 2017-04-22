### load from global
```
CS2R R9, SR_CLOCKLO;           
MOV R9, R9;                    
MOV R9, R9;                    // 2 mov to record clock
                               
I2I.S32.S32 R10, R8;           
SHR R12, R10, 0x1f;            
MOV R13, R12;                  
                               
MOV R12, R10;                  
MOV R12, R12;                  
MOV R13, R13;                  
                               
MOV R10, R12;                  
MOV R12, R13;                  
SHF.L.U64 R12, R10, 0x2, R12;  
                               
SHF.L.U64 R10, RZ, 0x2, R10;   
MOV R13, R2;                   
MOV R14, R3;                   
                               
IADD R10.CC, R13, R10;         
IADD.X R12, R14, R12;          
MOV R13, R12;                  
                               
MOV R12, R10;                  
MOV R12, R12;                  
MOV R13, R13;                  
                               
MOV R14, R12;                  
MOV R12, R13;                  
LEA R10.CC, R14, RZ;           
                               
LEA.HI.X P0, R12, R14, RZ, R12;
MOV R13, R12;                  
MOV R12, R10;                  
                               
LD.E R10, [R12], P0;           
MOV R12, R10;                  
CS2R R10, SR_CLOCKLO;         // 1 mov to recor clock 
```

3 mov (record clocks) + 25 (mov/lea/shf) + ld.e
= 28 * 15 + ld.e = 420 + ld.e = load 1 float to register  

### load from shared
inline "ld.shared.f32" doesn't work, use "ld.f32" instead

the compiler will convert the address to shared memory type, and use regular ld as shown before
```
// inline asm                                                               
mov.u64     %rd7, _Z18kernel_load_sharedPfPjS0_iff$__cuda_local_var_35442_32_non_const_sm;
cvta.shared.u64     %rd6, %rd7;                                             
// inline asm                                                               
ld.f32 %f3, [%rd6];
```

For a benchmarched sass as below
```
        /*0298*/                   CS2R R8, SR_CLOCKLO;             /* 0x50c8000005070008 */
                                                                    /* 0x00643c03fde01fef */
        /*02a8*/                   MOV R8, R8;                      /* 0x5c98078000870008 */
        /*02b0*/                   MOV R12, R8;                     /* 0x5c9807800087000c */
        /*02b8*/                   I2I.U32.U32 R8, RZ;              /* 0x5ce000000ff70a08 */
                                                                    /* 0x007fbc03fde01fef */
        /*02c8*/                   MOV R10, R8;                     /* 0x5c9807800087000a */
        /*02d0*/                   MOV R11, RZ;                     /* 0x5c9807800ff7000b */
        /*02d8*/                   MOV R10, R10;                    /* 0x5c98078000a7000a */
                                                                    /* 0x007fbc03fde01fef */
        /*02e8*/                   MOV R11, R11;                    /* 0x5c98078000b7000b */
        /*02f0*/                   MOV R13, R10;                    /* 0x5c98078000a7000d */
        /*02f8*/                   MOV R11, R11;                    /* 0x5c98078000b7000b */
                                                                    /* 0x007fbc03fde01fef */
        /*0308*/                   MOV R8, c[0x0][0x0];             /* 0x4c98078000070008 */
        /*0310*/                   MOV R10, RZ;                     /* 0x5c9807800ff7000a */
        /*0318*/                   LOP.OR R8, R13, R8;              /* 0x5c47020000870d08 */
                                                                    /* 0x007fbc03fde01fef */
        /*0328*/                   LOP.OR R10, R11, R10;            /* 0x5c47020000a70b0a */
        /*0330*/                   MOV R11, R10;                    /* 0x5c98078000a7000b */
        /*0338*/                   MOV R10, R8;                     /* 0x5c9807800087000a */
                                                                    /* 0x007fbc03fde01fef */
        /*0348*/                   MOV R10, R10;                    /* 0x5c98078000a7000a */
        /*0350*/                   MOV R11, R11;                    /* 0x5c98078000b7000b */
        /*0358*/                   MOV R13, R10;                    /* 0x5c98078000a7000d */
                                                                    /* 0x007fbc03fde01fef */
        /*0368*/                   MOV R10, R11;                    /* 0x5c98078000b7000a */
        /*0370*/                   LEA R8.CC, R13, RZ;              /* 0x5bd780000ff70d08 */
        /*0378*/                   LEA.HI.X P0, R10, R13, RZ, R10;  /* 0x5bd805400ff70d0a */
                                                                    /* 0x00643c03fde01fef */
        /*0388*/                   MOV R11, R10;                    /* 0x5c98078000a7000b */
        /*0390*/                   MOV R10, R8;                     /* 0x5c9807800087000a */
        /*0398*/                   LD.E R8, [R10], P0;              /* 0x8090000000070a08 */
                                                                    /* 0x007fbc03fde01fef */
        /*03a8*/                   MOV R8, R8;                      /* 0x5c98078000870008 */
        /*03b0*/                   CS2R R10, SR_CLOCKLO;            /* 0x50c800000507000a */

```

* (2 mov for start clock)
* 22 (i2i/mov/lop/lea) + ld.e 
* (1 mov for end clock)


