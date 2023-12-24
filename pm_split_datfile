#!/usr/bin/env python3

#################### ALESSANDRO RIDOLFI ########################

import sys
import os
import os.path
import glob
import numpy as np
import pylab as plt


class Inffile(object):
       def __init__(self, inffilename):
              inffile = open(inffilename, "r")
              for line in inffile:
                     if "Data file name without suffix" in line:
                            self.datafilebasenam = line.split("=")[-1].strip()
                     elif "Telescope used" in line:
                            self.telescope = line.split("=")[-1].strip()
                     elif "Instrument used" in line:
                            self.instrument = line.split("=")[-1].strip()
                     elif "Object being observed" in line:
                            self.source = line.split("=")[-1].strip()
                     elif "J2000 Right Ascension" in line:
                            self.RAJ = line.split("=")[-1].strip()
                     elif "J2000 Declination" in line:
                            self.DECJ = line.split("=")[-1].strip()
                     elif "Data observed by" in line:
                            self.observer = line.split("=")[-1].strip()
                     elif "Epoch of observation" in line:
                            self.start_MJD = np.float128(line.split("=")[-1].strip())
                     elif "Barycentered?" in line:
                            self.barycentered = int(line.split("=")[-1].strip())
                     elif "Number of bins in the time series" in line:
                            self.nsamples = int(line.split("=")[-1].strip())
                     elif "Width of each time series bin" in line:
                            self.tsamp_s = np.float128(line.split("=")[-1].strip())
                     elif "Any breaks in the data?" in line:
                            self.breaks_in_data = int(line.split("=")[-1].strip())
                     elif "Type of observation" in line:
                            self.obstype = line.split("=")[-1].strip()
                     elif "Beam diameter" in line:
                            self.beamdiameter = np.float128(line.split("=")[-1].strip())
                     elif "Dispersion measure" in line:
                            self.DM = np.float128(line.split("=")[-1].strip())
                     elif "Central freq of low channel" in line:
                            self.freq_ch1 = np.float128(line.split("=")[-1].strip())
                     elif "Total bandwidth" in line:
                            self.bw = np.float128(line.split("=")[-1].strip())
                     elif "Number of channels" in line:
                            self.nchan = int(line.split("=")[-1].strip())
                     elif "Channel bandwidth" in line:
                            self.bw_chan = np.float128(line.split("=")[-1].strip())
                     elif "Data analyzed by" in line:
                            self.analyzer = line.split("=")[-1].strip()
              inffile.close()


###################################################################################
# ARGOMENTI DA SHELL
string_version = "1.0-beta1 (23Oct2019)"
n_chunks = 0
chunk_length_s = 0
str_chunk_length_min = ""
str_split_type = ""

fraction_start = 0
fraction_end = 1

if (len(sys.argv) == 1 or ("-h" in sys.argv) or ("-help" in sys.argv) or ("--help" in sys.argv)):
       print("USAGE: %s -dat \"*.dat\"  {-n N_chunks   |   -length_s N   |  -start X.X -end Y.Y }" % (os.path.basename(sys.argv[0])))
       print()
       exit()
elif (("-version" in sys.argv) or ("--version" in sys.argv)):
       print("Version: %s" % (string_version))
       exit()
else:
       for j in range(1, len(sys.argv)):
              if (sys.argv[j] == "-dat"):
                     string_datfile = sys.argv[j+1]
                     if ("*" in string_datfile) or ("?" in string_datfile):
                            list_datfiles = sorted(glob.glob(string_datfile.strip("\"")))
                     else:
                            list_datfiles = string_datfile.split(",")
              elif (sys.argv[j] == "-n"):
                     str_split_type = "split_nchunks"
                     n_chunks = int(sys.argv[j+1])

              elif (sys.argv[j] == "-length_s"):
                     str_split_type = "split_length"
                     chunk_length_s = np.float64(sys.argv[j+1])
                     
              elif (sys.argv[j] == "-start"):
                     str_split_type = "split_start_end"
                     fraction_start = np.float64(sys.argv[j+1])
                     
              elif (sys.argv[j] == "-end"):
                     str_split_type = "split_start_end"
                     fraction_end = np.float64(sys.argv[j+1])


N_datfiles = len(list_datfiles)


for i in range(N_datfiles):
       datfile_name = list_datfiles[i]
       datfile_basename = os.path.splitext(os.path.basename(datfile_name))[0]
       inffile_name = datfile_name.replace(".dat", ".inf")
       inffile = Inffile(inffile_name)
       
       tsamp_s = inffile.tsamp_s
       nsamples = inffile.nsamples
       obs_length_s = tsamp_s * nsamples
       
       print("tsamp         = %.5f us" % (tsamp_s*1.e6))
       print("nsamples      = %d" % (nsamples))
       print("obs_length_s  = %.3f s" % (obs_length_s))
       
       
       if str_split_type == "split_start_end":
              
              delta_fraction = fraction_end - fraction_start
              print("delta_fraction = ", delta_fraction)
              numout = int(delta_fraction * nsamples)
              if numout % 2 != 0:
                     numout = numout + 1
              cmd_prepdata = "prepdata   -nobary   -dm 0   -start %s   -numout %d  -o %s_section%.2f-%.2f   %s" % (fraction_start, numout, datfile_basename, fraction_start, fraction_end, datfile_name)

              print(cmd_prepdata)
              os.system(cmd_prepdata)
              
       elif str_split_type == "split_nchunks" or str_split_type == "split_length":

              if str_split_type == "split_nchunks":
                     chunk_fraction_size = 1./n_chunks
                     numout_per_chunk = int(nsamples / np.float64(n_chunks) )
                     if numout_per_chunk % 2 != 0:
                            numout_per_chunk = numout_per_chunk + 1
                     chunk_length_s = numout_per_chunk * tsamp_s
                     
                     
              elif str_split_type == "split_length":
                     n_chunks = int(obs_length_s / chunk_length_s)
                     numout_per_chunk = int(chunk_length_s / np.float64(tsamp_s))
                     if numout_per_chunk % 2 != 0:
                            numout_per_chunk = numout_per_chunk + 1

                     chunk_fraction_size = chunk_length_s / obs_length_s
                     str_chunk_length_min = "%dm_" % (int(chunk_length_s / 60.))
              else:
                     print("ERROR! No mode was specified!")
                     exit()
                     
              if chunk_fraction_size < 1:
                     for i in range(n_chunks):
                            if n_chunks < 10:
                                   cmd_prepdata = "prepdata   -nobary   -dm 0   -start %.8f   -numout %d  -o %s_%sck%dof%d   %s"  % (i*chunk_fraction_size, numout_per_chunk, datfile_basename, str_chunk_length_min, i+1, n_chunks, datfile_name)
                            elif (n_chunks >= 10 and n_chunks <= 99):
                                   cmd_prepdata = "prepdata   -nobary   -dm 0   -start %.8f   -numout %d  -o %s_%sck%02dof%02d   %s"  % (i*chunk_fraction_size, numout_per_chunk, datfile_basename, str_chunk_length_min, i+1, n_chunks, datfile_name)
                            print(cmd_prepdata)
                            os.system(cmd_prepdata)
                            
              else:
                     print("Observation %s is shorter (%.2f s) than the requested chunk size (%.2f s). Skipping..." % (datfile_name, obs_length_s, chunk_length_s))
                     print()
                     print()
                     
