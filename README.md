运行命令
```
nohup run_breakseq2.py --reference hg19.fa --bams xxx.bam --work xxx --bwa /full_path/bwa \
      --samtools /full_path/samtools --bplib_gff /full_path/breakseq2_bplib_20150129_chr.gff \
      --nthreads 24 --sample xxx > xxx_breakseq2.log &
```


在使用过程中，发现报错信息

```
Traceback (most recent call last):
  File "/usr/bin/run_breakseq2.py", line 28, in <module>
    args.keep_temp, args.window, args.junction_length))
  File "/usr/lib/python2.7/site-packages/breakseq2/breakseq_top.py", line 96, in breakseq2_workflow
    breakseq_index.generate_bplib(bplib_gff, reference, bplib, junction_length)
  File "/usr/lib/python2.7/site-packages/breakseq2/breakseq_index.py", line 47, in generate_bplib
    flanks = sv.get_flanks()
  File "/usr/lib/python2.7/site-packages/breakseq2/biopy/io/SV.py", line 120, in get_flanks
    seqA=seqs.get_sequence(*seqs.get_window(self.name, self.start))
  File "/usr/lib/python2.7/site-packages/breakseq2/biopy/io/Fasta.py", line 20, in get_sequence
    seq = self.seqs[name].seq
KeyError: '1'
```

解决参考：(https://github.com/bioinform/breakseq2/issues/18)[https://github.com/bioinform/breakseq2/issues/18]

染色体编号chr1和1的区别
```
cat breakseq2_bplib_20150129.gff | awk -v OFS='\t' '$1="chr"$1' > breakseq2_bplib_20150129_chr.gff
sed -E 's/[0-9,X,Y].*/chr&/' breakseq2_bplib_20150129.ins > breakseq2_bplib_20150129_chr.ins
sed -E 's/[0-9,X,Y].*/chr&/' breakseq2_bplib_20150129.fna > breakseq2_bplib_20150129_chr.fna
```

解决之后发现另一个报错
```
ERROR 2021-12-03 21:44:13,138 preprocess_and_align-<Process(PoolWorker-8, started daemon)> Caught exception in worker thread
Traceback (most recent call last):
  File "/usr/lib/python2.7/site-packages/breakseq2/preprocess_and_align.py", line 37, in preprocess_and_align
    subprocess.check_call(bash_cmd, shell=True, stderr=logfd)
  File "/usr/lib64/python2.7/subprocess.py", line 542, in check_call
    raise CalledProcessError(retcode, cmd)
CalledProcessError: Command 'bash -c "bwa samse bplib.fa <(bwa aln bplib.fa 18.fq) 18.fq | samtools view -S - -1 -F 4 -bo 18.bam"' returned non-zero exit status 1
```

将`bwa samse bplib.fa <(bwa aln bplib.fa 18.fq) 18.fq | samtools view -S - -1 -F 4 -bo 18.bam`这个命令运行发现:

```
[bwa_aln] 17bp reads: max_diff = 2
[bwa_aln] 38bp reads: max_diff = 3
[bwa_aln] 64bp reads: max_diff = 4
[bwa_aln] 93bp reads: max_diff = 5
[bwa_aln] 124bp reads: max_diff = 6
[bwa_aln] 157bp reads: max_diff = 7
[bwa_aln] 190bp reads: max_diff = 8
[bwa_aln] 225bp reads: max_diff = 9
[bwa_aln_core] calculate SA coordinate... 5.27 sec
[bwa_aln_core] write to the disk... [W::sam_hdr_create] Duplicated sequence 'chr7:51356474-51356581:1KG_Phase1:Deletion:108:C'
[W::sam_hdr_create] Duplicated sequence 'chr20:62949734-62959201:1KG_Phase1:Deletion:9468:C'
0.03 sec
[bwa_aln_core] 262144 sequences have been processed.
[bwa_aln_core] calculate SA coordinate... 2.79 sec
[bwa_aln_core] write to the disk... [bwa_aln_core] convert to sequence coordinate... 0.04 sec
[bwa_aln_core] refine gapped alignments... 0.07 sec
[bwa_aln_core] print alignments... [E::sam_hrecs_update_hashes] Duplicate entry "chr7:51356474-51356581:1KG_Phase1:Deletion:108:C" in sam header
samtools view: failed to add PG line to the header
```

检查了bam文件header等等没发现问题，后面发现breakseq2_bplib_20150129_chr.gff文件有问题，
分别是3874行的7 51356474和8157行的20 62949734出现重复

直接vim命令行模式跳转到该位置，再使用编辑模式删除即可







