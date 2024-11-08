# pkg/xsysinfo/cpu.go  
## Package: xsysinfo  
  
### Imports:  
- sort  
- github.com/jaypipes/ghw  
- github.com/klauspost/cpuid/v2  
  
### External Data, Input Sources:  
- CPU information from the system  
  
### CPUCapabilities()  
This function returns a list of CPU capabilities as strings. It first retrieves CPU information using the ghw.CPU() function. Then, it iterates through the processors and their capabilities, adding each capability to a map. Finally, it converts the map to a list of strings, sorts the list, and returns it.  
  
### HasCPUCaps()  
This function checks if the CPU supports a given set of CPUID feature IDs. It uses the cpuid.CPU.Supports() function to determine if the CPU supports the specified feature IDs.  
  
### CPUPhysicalCores()  
This function returns the number of physical CPU cores. It first checks if the cpuid.CPU.PhysicalCores value is 0. If it is, it returns 1, otherwise, it returns the value of cpuid.CPU.PhysicalCores.  
  
  
  
# pkg/xsysinfo/gpu.go  
## Package: xsysinfo  
  
### Imports:  
- github.com/jaypipes/ghw  
- github.com/jaypipes/ghw/pkg/gpu  
  
### External Data, Input Sources:  
- ghw.GPU() function from the github.com/jaypipes/ghw package.  
  
### Summary:  
#### GPUs() function:  
This function retrieves a list of graphics cards from the system. It first calls the ghw.GPU() function to get the GPU information. If there is an error, it returns an error. Otherwise, it returns a slice of graphics cards from the GPU object.  
  
