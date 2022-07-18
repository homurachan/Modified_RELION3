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

# Modified RELION3.1.2

Download modi_relion-ver3.12_special_for_nonalign_polish_frame_recons.zip.

There are two new options on relion_motion_refine/mpi, which are "--read_from_file" and "--Not_mult_freqweight".

When calculating critical dose, per-frame reconstruction is a good way. Like relion_polish, per-frame reconstruction requires no modification to per-particle shift aka "tracks.star" and no change to weighting files aka "FCC_cc/w0/w1.mrc". 
If one wants to perform the per-frame reconstructions, the normal relion_polish should be run first. After 3D-refinement, the newer result we called "Refine3D/job021/run_data.star". Should not be confused by the older result used for polish, which we called "Refine3D/job010/run_data.star"
1. Let's call the result folder "Job001/micrographs". Afterwards, use `rm Job001/micrographs/*shiny*` to delete all shiny files. The folder Job001 will be copied several times, renaming them for better mark. e.g. `cp Job001 new_frame10`.
3. copy the command that in the "Job001/note.txt", which looks like `relion_motion_refine_mpi --i Refine3D/job010/run_data.star --f PostProcess/job011/postprocess.star --corr_mic MotionCorr/job000/corrected_micrographs.star --first_frame 1 --last_frame -1 --o Polish/Job001/ --s_vel 0.2 --s_div 5000 --s_acc 2 --combine_frames --bfac_minfreq 20 --bfac_maxfreq -1 --j 10 --pipeline_control Polish/Job001/`
4. Convert the command above to `mpirun -n $proc relion_motion_refine_mpi --i Refine3D/job010/run_data.star --f PostProcess/job011/postprocess.star --corr_mic MotionCorr/job000/corrected_micrographs.star --first_frame 10 --last_frame 10 --o Polish/new_frame10/ --combine_frames --j 10 --read_from_file --Not_mult_freqweight`. and run it. The particles it produced contained only the 10th frame. 
5. `cp Refine3D/job021/run_data.star frame10_run_data.star`. use text edit software to replace all the "rlnImageName" information from older  "XXXX@Polish/Job001/micrographs/MG_NNNN_shiny.mrcs" to "XXXX@Polish/new_frame10/micrographs/MG_NNNN_shiny.mrcs" on frame10_run_data.star
6. Use `mpirun -n $proc relion_reconstruct_mpi --i frame10_run_data.star --o frame10_run_data_half1/2.mrc --ctf --subset 1/2 --sym ANYSYMMETRY` to get final unfil-half map.
7. Change the --first_frame and --last_frame to your desired frame number and repeat step 4 to 6.

Note: --read_from_file can only be used if the folder "Job001/micrographs/" has "tracks.star FCC_cc.mrc FCC_w0.mrc FCC_w1.mrc".

When applying --Not_mult_freqweight, no frequency weight which stores in FCC_??.mrc would be used. It is essential not to apply freqweight while calculating critical dose. However, when measuring other SNR params, applying freqweight is recommanded. 
