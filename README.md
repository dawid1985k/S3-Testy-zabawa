# S3-Testy-zabawa
uses crt;
var v: byte;
i:integer;
Fl:File;
b:byte;
   fi:Text;
const
  S3_CUR_X        = $82E8;  {CUR Y}
  S3_CUR_Y        = $86E8;  {CUR X}
  S3_CUR_X2       = $82EA;  {CUR Y2}
  S3_CUR_Y2       = $86EA;  {CUR X2}
  S3_DEST_Y       = $8AE8;  {DESTY}
  S3_DEST_X       = $8EE8;  {DESTX}
  S3_DEST_Y2      = $8AEA;  {DESTY2}
  S3_DEST_X2      = $8EEA;  {DESTX2}
  S3_ERR_TERM     = $92E8;
  S3_ERR_TERM2    = $92EA;
  S3_CMD          = $9AE8;
  S3_CMD2         = $9AEA;
  S3_SHORT_STROKE = $9EE8;
  S3_BKGD_COLOR   = $A2E8;
  S3_FRGD_COLOR   = $A6E8;
  S3_WRT_MASK     = $AAE8;
  S3_RD_MASK      = $AEE8;
  S3_COLOR_CMP    = $B2E8;
  S3_BKGD_MIX     = $B6E8;
  S3_FRGD_MIX     = $BAE8;
  S3_SCISSOR_T    = $BEE8;
  S3_SCISSOR_L    = $BEE8;
  S3_SCISSOR_B    = $BEE8;
  S3_SCISSOR_R    = $BEE8;
  S3_PIX_CNTL     = $BEE8;
  S3_MULT_MISC    = $BEE8;
  S3_MIN_AXIS     = $96EA;
  S3_MAJ_AXIS     = $96E8;
  S3_MAJ_AXIS2    = $96EA;
  S3_PIX_TRANS    = $E2E8;
  S3_PIX_TRANS2   = $E2EA;
  S3_PAT_Y        = $EAE8;
  S3_PAT_X        = $EAEA;
function ReadCR(reg: byte):byte;
 begin
  Port[$3D4]:=reg;
  ReadCR:= Port[$3D5];
 end;
function ByteToHex(b :byte):string;
const
 hex: array[0..15] of char = ('0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F');
begin
 ByteToHex := hex[b shr 4] + hex[b and $0F];
end;
function HexStr(Value:Word; Digits :byte):string;
const
 hex: array[0..15] of char = ('0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F');
var s:string;
    i:byte;
begin
 s:='';
 for i:=1 to Digits do
  begin
   S:=hex[Value and $F] + S;
   Value:=value shr 4;
  end;
  HexStr:=s;
end;
procedure LogReg(const Name:String; IndexPort,DataPort: Word; Reg: Byte; var F:Text);
 var
  x:byte;
 begin
  Port[IndexPort]:=Reg;
  v:= Port[DataPort];
  Writeln(F, Name,'=',ByteToHex(v));
 end;
procedure LogRegW(const Name:String; PortAddr: Word; var F:Text);
 var
  v: Word;
   begin
    v:=PortW[PortAddr];
    Writeln(F,Name,' = ',HexSTR(v,4));
   end;

procedure OutW(port,value:Word);
 begin
  asm
   mov dx, port
   mov ax, value
   out dx,ax
  end;
 end;
procedure SetMode(mode:word);assembler;
 asm
  mov ax,4f02h
  mov bx,203h
  int $10
end;
procedure S3Putpixel(x,y:word; color:byte);
 var ofs:longint;

 bank: word;
 begin
   ofs:=longint(y)*800 + x;
   bank:=ofs shr 16;
  Port[$3D4]:=$6A;
  Port[$3D5]:=bank and $3F;
   ofs := ofs and $FFFF;
   Mem[$A000:ofs]:=color;
 end;
{Rysowanie prostokata wypelnionego kolorem - testy OK}
procedure S3_RectFill(x,y,w,h:word; color:byte);
 var cmd:word;
begin
 OutW(S3_FRGD_MIX,$0027);
 OutW(S3_FRGD_COLOR,color);
 OutW(S3_PIX_CNTL,$A000);
 OutW(S3_CUR_X,x);
 OutW(S3_CUR_Y,y);
 OutW(S3_MIN_AXIS,h-1);
 OutW(S3_MAJ_AXIS,w-1);
 OutW(S3_CMD,$40B1);
end;
begin
 asm
  {Chip wake up}
  mov dx,46E8h
  mov al,10h
  out dx,al

  mov dx,102h
  mov al,01h
  out dx,al

  mov dx,46E8h
  mov al,08h
  out dx,al

  mov dx,4AE8h
  xor al,al
  out dx,al
  {CR 38 - register on}
  mov dx,3D4h
  mov ax,4838h
  out dx,ax
  {CR 39 = register on}
  mov ax,0A539h
  out dx,ax
  {Odblokowanie zapisu CR0-CR7 (CR11 bit 7=0}
  mov dx,3D4h
  mov al,11h
  out dx,al
  inc dx
  in al,dx
  and al,7Fh
  out dx,al
  dec dx

  {SR8}
  mov dx,3C4h
  mov ax,0608h
  out dx,ax
  {CR40 - register on}
  mov dx,3D4h
  mov al,40h
  out dx,al
  inc dx
  in al,dx
  or  al,1
  out dx,al
  dec dx
  {CR51}
  mov dx,3D4h
  mov ax,0051h
  out dx,ax
  {Enhanced mode}
  mov dx,4AE8h
  mov ax,0001h
  out dx,ax
  {Screen on}
  {SR1 bit 5 = 1 {Screen off}
   mov dx, 3C4h
   mov al,01h
   out dx,al
   inc dx
   in al,dx
   or al,20h
   out dx,al
   dec dx
  {CR 17:bit 7 = 0 (Sync off}
   mov dx,3D4h
   mov al,17h
   out dx,al
   inc dx
   in al,dx
   and al,7Fh
   out dx,al
   dec dx
  {CR 31 Konfiguracja rejestrow rozszerzonych }
   mov dx,3D4h
   mov ax,8B31h
   out dx,ax
   mov ax,353Ah
   out dx,ax
   mov ax,0067h
   out dx,ax
   mov ax,105Dh
   out dx,ax
   mov ax,005Eh
   out dx,ax
  {CR 55}
   mov dx,3D4h
   mov ax,0155h
   out dx,ax
  {Ustawienie zegara PLL}
   mov dx,3C4h
   mov ax,0608h
   out dx,ax
   mov ax,4212h
   out dx,ax
   mov ax,2B13h
   out dx,ax
 {  mov ax,0018h
   out dx,ax }
   mov al,15h
   out dx,al
   inc dx
   in al,dx
   or al,20h
   out dx,al
   and al,0DFh
   out dx,al
   dec dx
  {Ustawienie rejestr CRTC}
   mov dx,3D4h
   mov ax,7F00h
   out dx,ax
   mov ax,6301h
   out dx,ax
   mov ax,6302h
   out dx,ax
   mov ax,0003h
   out dx,ax
   mov ax,6604h
   out dx,ax
   mov ax,1005h
   out dx,ax
   mov ax,6F06h
   out dx,ax
   mov ax,0E007h
   out dx,ax
   mov ax,0008h
   out dx,ax
   mov ax,2009h
   out dx,ax
   mov ax,5910h
   out dx,ax
   mov ax,0E11h
   out dx,ax
   mov ax,5712h
   out dx,ax
   mov ax,6413h
   out dx,ax
   mov ax,0014h
   out dx,ax
   mov ax,5815h
   out dx,ax
   mov ax,7116h
   out dx,ax
   {Ustawienie sequencera}
   mov dx, 3C4h
   mov ax,0101h
   out dx,ax
   mov ax,0F02h
   out dx,ax
   mov ax,0E04h
   out dx,ax
   {Konfiguracja grpihic coltrolle}
   mov dx,3CEh
   mov ax,4005h
   out dx,ax
   mov ax,0D06h
   out dx,ax
   {Atribut controler}
   mov dx,3DAh
   in al,dx
   mov dx,3C0h
   mov al,10h
   out dx,al
   mov al,41h
   out dx,al
   mov al,13h
   out dx,al
   mov al,00h
   out dx,al
   mov dx,3DAh
   in al,dx
   mov dx,3C0h
   mov al,20h
   out dx,al
   {Mix output register}
   mov dx,3C2h
   mov al,2Ch
   out dx,al
   {PEL Mask}
   mov dx,3C6h
   mov al,0FFh
   out dx,al

  {Screen on}
  {CR17:bit 1 =1 Sync on}
   mov dx,3D4h
   mov al,17h
   out dx,al
   inc dx
   in al,dx
   and al,7Fh
   out dx,al
   dec dx
   {SR1}
   mov dx,3C4h
   mov al,01h
   out dx,al
   inc dx
   in al,dx
   and al,0DFh
   out dx,al
 end;
S3Putpixel(100,100,20);
delay(1000);
readln;
{ S3_RectFill(500,500,100,100,15);
 delay(1000);                        }
 Assign(fi,'C:\S3LOG.TXT');
 Rewrite(fi);

 LogReg('CR00',$3D4,$3D5,$00,fi);
 LogReg('CR01',$3D4,$3D5,$01,fi);
 LogReg('CR02',$3D4,$3D5,$02,fi);
 LogReg('CR03',$3D4,$3D5,$03,fi);
 LogReg('CR04',$3D4,$3D5,$04,fi);
 LogReg('CR05',$3D4,$3D5,$05,fi);
 LogReg('CR06',$3D4,$3D5,$06,fi);
 LogReg('CR07',$3D4,$3D5,$07,fi);
 LogReg('CR08',$3D4,$3D5,$08,fi);
 LogReg('CR09',$3D4,$3D5,$09,fi);
 LogReg('CR0A',$3D4,$3D5,$0A,fi);
 LogReg('CR0B',$3D4,$3D5,$0B,fi);
 LogReg('CR0C',$3D4,$3D5,$0C,fi);
 LogReg('CR0D',$3D4,$3D5,$0D,fi);
 LogReg('CR0E',$3D4,$3D5,$0E,fi);
 LogReg('CR0F',$3D4,$3D5,$0F,fi);

 LogReg('CR10',$3D4,$3D5,$10,fi);
 LogReg('CR11',$3D4,$3D5,$11,fi);
 LogReg('CR12',$3D4,$3D5,$12,fi);
 LogReg('CR13',$3D4,$3D5,$13,fi);
 LogReg('CR14',$3D4,$3D5,$14,fi);
 LogReg('CR15',$3D4,$3D5,$15,fi);
 LogReg('CR16',$3D4,$3D5,$16,fi);
 LogReg('CR17',$3D4,$3D5,$17,fi);
 LogReg('CR18',$3D4,$3D5,$18,fi);
 LogReg('CR19',$3D4,$3D5,$19,fi);
 LogReg('CR1A',$3D4,$3D5,$1A,fi);
 LogReg('CR1B',$3D4,$3D5,$1B,fi);
 LogReg('CR1C',$3D4,$3D5,$1C,fi);
 LogReg('CR1D',$3D4,$3D5,$1D,fi);
 LogReg('CR1E',$3D4,$3D5,$1E,fi);
 LogReg('CR1F',$3D4,$3D5,$1F,fi);

 LogReg('CR20',$3D4,$3D5,$20,fi);
 LogReg('CR21',$3D4,$3D5,$21,fi);
 LogReg('CR22',$3D4,$3D5,$22,fi);
 LogReg('CR23',$3D4,$3D5,$23,fi);
 LogReg('CR24',$3D4,$3D5,$24,fi);
 LogReg('CR25',$3D4,$3D5,$25,fi);
 LogReg('CR26',$3D4,$3D5,$26,fi);
 LogReg('CR27',$3D4,$3D5,$27,fi);
 LogReg('CR28',$3D4,$3D5,$28,fi);
 LogReg('CR29',$3D4,$3D5,$29,fi);
 LogReg('CR2A',$3D4,$3D5,$2A,fi);
 LogReg('CR2B',$3D4,$3D5,$2B,fi);
 LogReg('CR2C',$3D4,$3D5,$2C,fi);
 LogReg('CR2D',$3D4,$3D5,$2D,fi);
 LogReg('CR2E',$3D4,$3D5,$2E,fi);
 LogReg('CR2F',$3D4,$3D5,$2F,fi);

 LogReg('CR30',$3D4,$3D5,$30,fi);
 LogReg('CR31',$3D4,$3D5,$31,fi);
 LogReg('CR32',$3D4,$3D5,$32,fi);
 LogReg('CR33',$3D4,$3D5,$33,fi);
 LogReg('CR34',$3D4,$3D5,$34,fi);
 LogReg('CR35',$3D4,$3D5,$35,fi);
 LogReg('CR36',$3D4,$3D5,$36,fi);
 LogReg('CR37',$3D4,$3D5,$37,fi);
 LogReg('CR38',$3D4,$3D5,$38,fi);
 LogReg('CR39',$3D4,$3D5,$39,fi);
 LogReg('CR3A',$3D4,$3D5,$3A,fi);
 LogReg('CR3B',$3D4,$3D5,$3B,fi);
 LogReg('CR3C',$3D4,$3D5,$3C,fi);
 LogReg('CR3D',$3D4,$3D5,$3D,fi);
 LogReg('CR3E',$3D4,$3D5,$3E,fi);
 LogReg('CR3F',$3D4,$3D5,$3F,fi);

 LogReg('CR40',$3D4,$3D5,$40,fi);
 LogReg('CR41',$3D4,$3D5,$41,fi);
 LogReg('CR42',$3D4,$3D5,$42,fi);
 LogReg('CR43',$3D4,$3D5,$43,fi);
 LogReg('CR44',$3D4,$3D5,$44,fi);
 LogReg('CR45',$3D4,$3D5,$45,fi);
 LogReg('CR46',$3D4,$3D5,$46,fi);
 LogReg('CR47',$3D4,$3D5,$47,fi);
 LogReg('CR48',$3D4,$3D5,$48,fi);
 LogReg('CR49',$3D4,$3D5,$49,fi);
 LogReg('CR4A',$3D4,$3D5,$4A,fi);
 LogReg('CR4B',$3D4,$3D5,$4B,fi);
 LogReg('CR4C',$3D4,$3D5,$4C,fi);
 LogReg('CR4D',$3D4,$3D5,$4D,fi);
 LogReg('CR4E',$3D4,$3D5,$4E,fi);
 LogReg('CR4F',$3D4,$3D5,$4F,fi);

 LogReg('CR50',$3D4,$3D5,$50,fi);
 LogReg('CR51',$3D4,$3D5,$51,fi);
 LogReg('CR52',$3D4,$3D5,$52,fi);
 LogReg('CR53',$3D4,$3D5,$53,fi);
 LogReg('CR54',$3D4,$3D5,$54,fi);
 LogReg('CR55',$3D4,$3D5,$55,fi);
 LogReg('CR56',$3D4,$3D5,$56,fi);
 LogReg('CR57',$3D4,$3D5,$57,fi);
 LogReg('CR58',$3D4,$3D5,$58,fi);
 LogReg('CR59',$3D4,$3D5,$59,fi);
 LogReg('CR5A',$3D4,$3D5,$5A,fi);
 LogReg('CR5B',$3D4,$3D5,$5B,fi);
 LogReg('CR5C',$3D4,$3D5,$5C,fi);
 LogReg('CR5D',$3D4,$3D5,$5D,fi);
 LogReg('CR5E',$3D4,$3D5,$5E,fi);
 LogReg('CR5F',$3D4,$3D5,$5F,fi);

 LogReg('CR60',$3D4,$3D5,$60,fi);
 LogReg('CR61',$3D4,$3D5,$61,fi);
 LogReg('CR62',$3D4,$3D5,$62,fi);
 LogReg('CR63',$3D4,$3D5,$63,fi);
 LogReg('CR64',$3D4,$3D5,$64,fi);
 LogReg('CR65',$3D4,$3D5,$65,fi);
 LogReg('CR66',$3D4,$3D5,$66,fi);
 LogReg('CR67',$3D4,$3D5,$67,fi);
 LogReg('CR68',$3D4,$3D5,$68,fi);
 LogReg('CR69',$3D4,$3D5,$69,fi);
 LogReg('CR6A',$3D4,$3D5,$6A,fi);
 LogReg('CR6B',$3D4,$3D5,$6B,fi);
 LogReg('CR6C',$3D4,$3D5,$6C,fi);
 LogReg('CR6D',$3D4,$3D5,$6D,fi);
 LogReg('CR6E',$3D4,$3D5,$6E,fi);
 LogReg('CR6F',$3D4,$3D5,$6F,fi);

 LogReg('CR70',$3D4,$3D5,$70,fi);
 LogReg('CR71',$3D4,$3D5,$71,fi);
 LogReg('CR72',$3D4,$3D5,$72,fi);
 LogReg('CR73',$3D4,$3D5,$73,fi);
 LogReg('CR74',$3D4,$3D5,$74,fi);
 LogReg('CR75',$3D4,$3D5,$75,fi);
 LogReg('CR76',$3D4,$3D5,$76,fi);
 LogReg('CR77',$3D4,$3D5,$77,fi);
 LogReg('CR78',$3D4,$3D5,$78,fi);
 LogReg('CR79',$3D4,$3D5,$79,fi);
 LogReg('CR7A',$3D4,$3D5,$7A,fi);
 LogReg('CR7B',$3D4,$3D5,$7B,fi);
 LogReg('CR7C',$3D4,$3D5,$7C,fi);
 LogReg('CR7D',$3D4,$3D5,$7D,fi);
 LogReg('CR7E',$3D4,$3D5,$7E,fi);
 LogReg('CR7F',$3D4,$3D5,$7F,fi);

 LogReg('CR35',$3D4,$3D5,$35,fi);
 LogReg('CR40',$3D4,$3D5,$40,fi);
 LogReg('CR31',$3D4,$3D5,$31,fi);
 LogReg('CR3A',$3D4,$3D5,$3A,fi);
 LogReg('CR67',$3D4,$3D5,$67,fi);
 LogReGW('42E8h',$42E8, fi);
 LogReGW('4AE8h',$4AE8, fi);
 LogReGW('82E8h',$82E8, fi);
 LogReGW('8AE8h',$8AE8, fi);
 Close(fi);
asm
 mov ax,0003h
 int 10h
end;
 Writeln('CR 38 = ',ByteToHex(ReadCR($38)));
 Writeln('CR 39 = ',ByteToHex(ReadCR($39)));
 Writeln('CR 40 = ',ByteToHex(ReadCR($40)));
 Writeln('CR 53 = ',ByteToHex(ReadCR($53)));
 Writeln('CR 09 = ',ByteToHex(ReadCR($09)));
 Writeln('CR 36 = ',ByteToHex(ReadCR($36)));
 Writeln('CR 37 = ',ByteToHex(ReadCR($37)));
 Writeln('CR 68 = ',ByteToHex(ReadCR($68)));
 Writeln('CR 33 = ',ByteToHex(ReadCR($33)));
 Writeln('CR 35 = ',ByteToHex(ReadCR($35)));
 Writeln('CR 13 = ',ByteToHex(ReadCR($13)));
 Writeln('CR 17 = ',ByteToHex(ReadCR($17)));
 Writeln('CR 31 = ',ByteToHex(ReadCR($31)));

{ setmode(1);  }
 Readln;
end.
