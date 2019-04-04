# MujocoSim.jl
Render code for the MuJoCo physics simulation software, in Julia.

# Usage
This is example code for how to render with MuJoCo.jl, with most of the structure and features taken from the simulate.cpp example code provided by the default [MuJoCo software](http://mujoco.org/). While some of the newest UI features are missing, the current code allows for passive dynamics simulation with mouse and camera interactivity.

```julia
using MujocoSim
MujocoSim.simulate("humanoid.xml") # where the file is a Mujoco XML file of your choice
```

This will use GLFW to open up an OpenGL window rendering the system / robot. The system starts paused; press `space` to begin time.

This works by having a large mjSim struct that holds the relevant rendering information, as well as MuJoCo model and data for simulation of the physics.

# Modification
Passive dynamics may be useful for testing out the xml model (althought using the orignal simulate.cpp is recommended), so this example code should be modified for your use case. To do so, one would need to create a render function, do whatever work is desired, and register the render function with GLFW.

```julia
function render(s::MujocoSim.mjSim, w::GLFW.Window)
   wi, hi = GLFW.GetFramebufferSize(w)
   rect = mjrRect(Cint(0), Cint(0), Cint(wi), Cint(hi))
   
   # Your work here
   # mj_step(s.m, s.d) will step forward time at the display framerate
   
   mjv_updateScene(s.m, s.d, s.vopt, s.pert, s.cam, Int(mj.CAT_ALL), s.scn)
   mjr_render(rect, s.scn, s.con)
   GLFW.SwapBuffers(w)
end
s = MujocoSim.start(mujoco_model, mujoco_data, window_width, window_height)
GLFW.SetWindowRefreshCallback(s.window, render) 
```
One could also modify the functions in line, or duplicate the code as desired. Your work may be to load data from a file and insert it into `s.d.qpos` and `s.d.qvel` to play back trajectories, or even take in the current observation state and pass it through a neural network policy to calculate controls that are inserted into `s.d.ctrl` before calling `mj_step` (although this would not be at the real-world time rate).

# Additional Information

Install `ffmpeg` if you'd like to render out videos by pressing `v` in the rendering window; this kind of assumes a Linux file system and dumps data to `/tmp/video.mp4`.

Any issues can be brought up in this repo or the [MuJoCo.jl](https://github.com/klowrey/MuJoCo.jl) repo for MuJoCo Julia wrapper bugs. I cannot fix problems with MuJoCo itself.
