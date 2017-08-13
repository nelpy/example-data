# Data formats for `linear-track`

## trajectory.videoPositionTracking
Binary file with header:
```
<Start settings>\n
threshold: 199\n
dark: 0\n
clockrate: 30000\n
camera resolution: 640x480\n
pixel scale: 0 pix/cm\n
Fields: <time uint32><xloc uint16><yloc uint16><xloc2 uint16><yloc2 uint16>\n
<End settings>\n
```
In this particular case, only a single LED was tracked, and therefore `xloc2` and `yloc2` are all zeroes.

## spikes.mat
Matlab file with a single field, namely `spikes`.

`spikes` is an ndarray of length 13, each of which corresponds to a tetrode that has been sorted with MatClust.

Within each tetrode array, there are a number of ndarrays with datatype `dtype=[('time', 'O'), ('timerange', 'O'), ('meanrate', 'O')]`. Some of these arrays could be empty, so we have to filter them further. Importantly though, each of the arrays could be considered a (possibly empty) unit, and the spike times are given in the `time` component of the array.

This spike file format is unfortunately needlessly complicated, and so we provide a simple snippet of code to parse the data into a more friendly format:
```python
spikes = []
ct = 0
num_array = 0
for ii, array in enumerate(mat['spikes']):
    # If empty array, that particular tetrode was not sorted
    if (array.size > 1):
        for jj, subarray in enumerate(array):
            if (subarray.size != 0):
                # Exclude tetrodes with no spikes
                if (len(subarray['time'].ravel()[0]) != 0):
                    spikes.append(subarray['time'].ravel()[0])
                    ct +=1
    elif (array.size == 1):
        if (len(array['time'].ravel()[0]) != 0):
            spikes.append(array['time'].ravel()[0])
            ct +=1
print("Found {} non-empty units total".format(ct))
```
After running this snippet on the `spike` data, we are left with a list of lists, where ith inner list is simply a list of all the spike times associated with the ith unit.
