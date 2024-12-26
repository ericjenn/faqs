# Modeling with FreeCad

## Generate the mesh

- Go to the mesh tool
- select a part and click on the part=>mesh button
- select your mesher and set the meshing options
- right click on the mesh and use the "export" option to save it as a STL file.

# 3D printing

## 3D printing with Trinus

- Save the mesh in the STL format (see [here](#generate-the-mesh)
- Import the STL file in Pango
- Slice it using Pango
- Save the gcode to the SDCARD
- Install the SDCARD in the printer
- Back in Pago
  - Set the bed temperature if needed (highmy recommanded). Use the Preferences/Settings menu, Advanced/Temperature item in the dialog box (see [picture](./imgs/trinus_bed_temp.jpg)). I have set it to 50°C and it works just fine.
  - Open the console (menu view/console).
  - Select the gcode to print
- That's it.

Note that, when the printer starts to print, it generates some filament that is likely to mess up the whole printing process if it sticks to the printer's head. So, be careful to remove it.
