# PUL
Pralka-automat stanów
----------------------------------------------------------------------------------
-- Company: WEMiF EiT
-- Engineer: Aleksandra Hel
-- 
-- Create Date:    18:21:49 05/31/2015 
-- Design Name: 
-- Module Name:    pralka_pul - pralka_arch 
-- Project Name: Pralka- projekt PUL
-- Target Devices: 
-- Tool versions: 
-- Description: 
--
-- Dependencies: 
--
-- Revision: 
-- Revision 0.01 - File Created
-- Additional Comments: 
--
----------------------------------------------------------------------------------
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.STD_LOGIC_unsigned.all;

-- Uncomment the following library declaration if using
-- arithmetic functions with Signed or Unsigned values
--use IEEE.NUMERIC_STD.ALL;

-- Uncomment the following library declaration if instantiating
-- any Xilinx primitives in this code.
--library UNISIM;
--use UNISIM.VComponents.all;

entity pralka_pul is
    Port ( clk : in  STD_LOGIC;
           reset : in  STD_LOGIC;
           prog : in  STD_LOGIC_VECTOR (1 downto 0);
           temp : in  STD_LOGIC_vector(1 downto 0);
			  woda_hi, woda_lo : in std_logic;  
           s_wyl : out  STD_LOGIC;
           s_woda : out  STD_LOGIC;
           s_prwst : out  STD_LOGIC;
           s_prwla : out  STD_LOGIC;
           s_odpomp : out  STD_LOGIC;
           s_wir : out  STD_LOGIC;
			  gotowe : out  STD_LOGIC);
end pralka_pul;

architecture pralka_arch of pralka_pul is

type STANY is (STOP, WODA, PRANIE_WSTEPNE, PRANIE_WLASC, ODPOMP, WIROWANIE);
signal stan,stan_nast: STANY;
signal licznik_czasu : std_logic_vector(3 downto 0);
signal licznik_plukan : std_logic_vector(2 downto 0);

begin


reg: process (reset, clk)	
	begin
		if reset ='1' then
			stan <= STOP;
		elsif clk'event and clk='1' then
			stan <= stan_nast;		
		end if;
end process reg;

dzialanie: process(stan, prog, licznik_plukan, licznik_czasu) 

begin

stan_nast <= stan;	

case stan is

when STOP =>
				if prog="01" and licznik_plukan = "000" then	
				stan_nast <= WODA;
				elsif prog="10" then 
				stan_nast<= WIROWANIE;
				end if;
				
when WODA => 
				if prog="00"  and licznik_plukan/= "000" then 
				stan_nast <= ODPOMP;
				elsif woda_hi ='1' then
				if prog="01" then 
				stan_nast <= PRANIE_WSTEPNE;
				elsif prog="11" then 
				stan_nast <= PRANIE_WLASC;
				end if;
				end if;
				
when PRANIE_WSTEPNE=>
				if prog="00" then 
				stan_nast <= ODPOMP;
				elsif prog="01" and temp/="00" then
				stan_nast <= PRANIE_WLASC;
				end if;
				
when PRANIE_WLASC=> --stan_nast <= ODPOMP;
				if prog="00" then 
				stan_nast <= ODPOMP;
				elsif prog="01" then 
				stan_nast <= ODPOMP;
				end if;
				
when ODPOMP =>
				if prog="00" then stan_nast <= STOP;
				elsif (prog="01" and woda_lo='1') then
				--elsif prog="01" then
				--	if licznik_plukan = "011" then 
						stan_nast <= WIROWANIE;
			--		else stan_nast <= WODA;
			--		end if;
				end if;
when others=> 
				stan_nast <= WODA;

end case;

end process dzialanie;


licznik_czasu_prania: process (reset, clk)
begin
	if reset = '0' then	
		licznik_czasu <= (others => '0');	
	elsif clk'event and clk='1' then
		if stan = PRANIE_WLASC then
			licznik_czasu <= licznik_czasu + "01";
		else
			licznik_czasu <= (others => '0');
		end if;
	end if;
end process licznik_czasu_prania;


licznik_ilosci_plukan: process (reset, clk)
begin
	if reset = '0' then	
		licznik_plukan <= (others => '0');	
	elsif clk'event and clk='1' then
		if stan = ODPOMP and stan_nast = WODA then
			licznik_plukan <= licznik_plukan+"01";
		else
			licznik_plukan <= licznik_plukan;
		end if;
	end if;
end process licznik_ilosci_plukan;

s_wyl <= '1' when stan=STOP else '0';
s_woda  <= '1' when stan=WODA else '0';
s_prwst <= '1' when stan=PRANIE_WSTEPNE else '0';
s_prwla <= '1' when stan = PRANIE_WLASC else '0';
s_odpomp <= '1' when stan = ODPOMP else '0';
s_wir <= '1' when stan = WIROWANIE else '0';
gotowe <= '1' when stan = WIROWANIE --and licznik_plukan= "011" 
and woda_lo = '1' else '0';



end pralka_arch;

