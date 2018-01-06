# GITHUB-project-APB
Verification of AMBA APB Protocol
APB is a low bandwidth and low performance bus.
Because of 
1. low power consumption
2. reduces interface complexity
3. unpipelined protocol, so interfaces with low bandwidth peripherals.

The bridge connects the high performance AHB or ASB bus to APB bus. So for APB the bridge acts as master and all devices connected to APB bus acts as slaves.
Test Plan:
From low level components implement the APB sequence item which generates random read and write transactions and the sequencer passes it to the driver through TLM ports. And the Master driver will convert those into pin level information and drive on to the APB Interface. The Monitor looks at the interface level toggling and translates them into APB Transactions which can be used by any scoreboard. And the agent component which encapsulates all the components. In this we connect Driver and Sequencer in connect phase.So we have a master driver that understands a protocol.

** Implementing transaction: 
For creating a transaction, Create an APB Transaction class (which I call as APB read/write class). It must have data members Command type, Address, write data, read data.inheriting from uvm_sequence_item base class. 

* Implementing Driver:
Coming to implementing the driver, it is derived from uvm_driver base class and parameterized with apb_rw. 
       In build_phase() we get the handle virtual interface from agent or from uvm_config database. Also we implement the read and write tasks here.

In run_phase() we call a get_next_item() to get an item from sequencer. Once we get an item we notify to the sequencer that we received the transaction by calling item_done. (Handshaking between Driver- Sequencer)
 
** In Main test component, we set the physical interface handle in uvm_config_db using set static method and use this interface handle in driver, agent and sequencer.
       Next, we call the test by passing run_test method. 

-> Slave:
Input control signals - Pclk, Pselect,Preset,Penable,Pwrite
Inputs - Paddr, Pwdata(32 bits) from bridge.
Outputs- Prdata (32 bit read data bus)

Totally 3 states to operate: IDLE, SETUP, ACCESS
IDLE: Psel =0, Penable=0,(no operation)

SETUP state: Psel=1, Penable =0 ( Pwrite, Paddr, Pwdata are also provided during this phase) 
   - remains in this phase for one clock cycle and shift to access phase for next rising clock edge.

Access phase: Penable =1, Psel=1
 * In read operation - Prdata is present on the bus and Penable remains High in one clock cycle. If no data transfer is reqired, the bus will move to IDLE state.

*Write cycle: Psel, Pwrite, Paddr, Pwdata are active- T1 clock edge- SETUP cycle. At T2 Penable, Pready are active- ACCESS phase. At T3 Penable =0
