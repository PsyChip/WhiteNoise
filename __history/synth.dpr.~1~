program synth;

uses
  Windows,
  Bass in '../bass.pas';

const
  PI = 3.14159265358979323846;
  TABLESIZE = 2048;
  MAXVOL = 9000;
  KEYS = 20;

var
  info: BASS_INFO;
  SineTable: array[0..TABLESIZE - 1] of Integer;
  aVol: array[0..KEYS - 1] of Integer =(0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0);
  aPos: array[0..KEYS - 1] of Integer = (0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0);

function IntPower(const Base: Extended; const Exponent: Integer): Extended;
asm
        mov     ecx, eax
        cdq
        fld1                      { Result := 1 }
        XOR     eax, edx
        sub     eax, edx          { eax := Abs(Exponent) }
        jz      @@3
        fld     Base
        jmp     @@2

@@1:    fmul    ST, ST            { X := Base * Base }

@@2:    shr     eax, 1
        jnc     @@1
        fmul    ST(1), ST          { Result := Result * X }
        jnz     @@1
        fstp    st                { pop X from FPU stack }
        cmp     ecx, 0
        jge     @@3
        fld1
        fdivrp                    { Result := 1 / Result }

@@3:
        fwait
end;

//---------------------------------------------------------

function Power(const Base, Exponent: Extended): Extended;
begin
  if Exponent = 0.0 then
    Result := 1.0               { n**0 = 1 }
  else if (Base = 0.0) and (Exponent > 0.0) then
    Result := 0.0               { 0**n = 0, n > 0 }
  else if (Frac(Exponent) = 0.0) and (Abs(Exponent) <= MaxInt) then
    Result := IntPower(Base, Integer(Trunc(Exponent)))
  else
    Result := Exp(Exponent * Ln(Base))
end;

function WriteStream(Handle: HSTREAM; Buffer: Pointer; Len: DWORD; User: Pointer): DWORD; stdcall;
type
  BufArray = array[0..0] of SmallInt;
var
  I, J, K: Integer;
  f: Single;
  Buf: ^BufArray absolute Buffer;
begin
  FillChar(Buffer^, Len, 0);
  for I := 0 to KEYS - 1 do
  begin
    if aVol[I] = 0 then
      Continue;
    f := Power(2.0, (I + 3) / 12.0) * TABLESIZE * 440.0 / info.freq;
    for K := 0 to (Len div 4 - 1) do
    begin
      if aVol[I] = 0 then
        Continue;

      inc(aPos[I]);
      J := Round(SineTable[Round(aPos[I] * f) and pred(TABLESIZE)] * aVol[I] / MAXVOL);
      inc(J, Buf[K * 2]);
      if J > 32767 then
        J := 32767
      else if J < -32768 then
        J := -32768;
      // left and right channels are the same
      Buf[K * 2 + 1] := J;
      Buf[K * 2] := J;
      if aVol[I] < MAXVOL then
        dec(aVol[I]);
    end;
  end;
  Result := Len;
end;

//---------------------------------------------------------

var
  Stream: HSTREAM;
  KeyIn: INPUT_RECORD;
  bKey: Integer;
  I, BufLen: DWORD;
  J: HFX;
  St: string;


begin
  BASS_SetConfig(BASS_CONFIG_UPDATEPERIOD, 10);
  if not BASS_Init(-1, 44100, BASS_DEVICE_LATENCY, 0, NIL) then exit;
  for I := 0 to TABLESIZE - 1 do SineTable[I] := Round((sin(2.0 * PI * I / TABLESIZE) * 7000.0));
  BASS_GetInfo(info);
  BASS_SetConfig(BASS_CONFIG_BUFFER, 64 + info.minbuf);
  BufLen := BASS_GetConfig(BASS_CONFIG_BUFFER);
  if info.freq = 0 then info.freq := 44100;
  Stream := BASS_StreamCreate(info.freq, 2, 0, @WriteStream, NIL);
  BASS_ChannelPlay(Stream, False);

  var Rand: integer;
 while true do begin
 Rand:=Random(19);
    aPos[Rand] := 0;
	  aVol[Rand] := 25;
   sleep(1);
 end;


  BASS_Free;
end.

