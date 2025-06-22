# Design_compile.tcl
set design_name "chip_top"
set clk_port "clk"
set clk_period 1.0

# Librart Setup
set_app_var search_path [list ./libs ./rtl]
set_app_var target_library [list "typical.db"]
set_app_var link_library "* typical.db"

# Read and Elaborate
read_verilog -autoread
analyze -foramat verilog [glob ./rtl/*.v]
elaborate $design_name
current_design $design_name

# Clock and Timing Setup
create_clock -name $clk_port -period $clk_period [get_ports $clk_port]
set_clock_uncertainty 0.1 [get_clocks $clk_port]

set_input_delay 0.2 -clock $clk_port [remove_from_collection [all_inputs] [get_ports $clk_port]]
set_output_delay 0.2 -clock $clk_port [all_output]

# Constraints
set_max_area 0
set_max_fanout 10
set_max_transition 0.5
set_max_capacitance 0.2

# Optimization
compile_ultra -gate_clock
report_timing -sort_by group -delay max -path full_clock_expanded > reports/post_synth_timing.rpt
report_area > reports/post_synth_area.rpt

# Save Design
write -format ddc -hierarchy -output $design_name.ddc
write_sdc $design_name.sdc
