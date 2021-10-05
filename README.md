# Modified_RELION3
Modification version of RELION3 beta/3.0.8/3.1, for non-alignment polish process.

There are two zip files: modi_relion-3.0_beta_special_for_nonalign_polish_frame_recons_with_bfweighting.zip and modi_relion-ver3.1_special_for_nonalign_polish_frame_recons.zip, which is modified from RELION3.0 and 3.1. relion-ver3.1_new_polish_nomotion.zip is modified for using MotionCorr star files as tracks for polish.
unzip corresponding files and compile those just like conventional RELION.

In order to process non-alignment polish, following steps are recommanded.
1. you should perform polish firstly as usual. A folder like "Polish/job???" is required. The tracks and FCC weights will be kept and read. You can refine the polish particles and the refined star-file "REFINE_run_data.star" can be used later.
2. Run `rm *shiny*` command on folder containing all "*tracks.star".
3. `cp Polish/job??? Polish/destination`
4. Use "print command" in RELION GUI to check the parameters on polish command. Mark your particles/postprocess/motioncorr star filename.
5. Such command `nohup mpirun -n N `which relion_motion_refine_mpi` --i INPUT_PARTICLE.star --f INPUT_PARTICLE_POSTPROCESS.star --corr_mic MotionCorr/jobxxx/corrected_micrographs.star --m1 INPUT_PARTICLE_POSTPROCESS_half1.mrc --m2 INPUT_PARTICLE_POSTPROCESS_half2.mrc --mask YOUR_MASK.mrc --first_frame 1 --last_frame -1 --o Polish/destination/ --combine_frames --j NN > output.log &` is suggested.
6. After running command of step 5, re-naming image names in "REFINE_run_data.star" so that it points to particles in Polish/destination.
7. Reconstructed using `relion_reconstruct --i REFINE_run_data_modified.star --o half1/2.mrc --subset 1/2 --sym YOUR_SYM --ctf`.
