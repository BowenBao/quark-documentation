Language Model Quantization using Quark
=======================================

This document provides examples of quantizing and exporting the language models(OPT, Llama…) using Quark. We offer several scripts for various quantization strategies. If users wish to apply their own **Customer Settings** for the ``calibration method``, ``dataset``, or ``quant config``, they can refer to the Quark's User Guide for help.

Models supported
----------------

+--------------+-------------------------+---------+---------+--------------------+-------------------------------------+---------+-------------+------------------------+------------------+
| Model Family | Model Name              |  FP16   | BFP16   | FP8(+FP8_KV_CACHE) | W_UIN4(Per group)+A_BF16(+AWQ/GPTQ) | INT8    | SmoothQuant | FP8 SafeTensors Export | INT8 ONNX Export |
+==============+=========================+=========+=========+====================+=====================================+=========+=============+========================+==================+
| LLAMA 2      | meta-llama/Llama-2-*-hf |  ✓      | ✓       | ✓                  | ✓                                   | ✓       | ✓           | ✓                      | ✓                |
+--------------+-------------------------+---------+---------+--------------------+-------------------------------------+---------+-------------+------------------------+------------------+
| LLAMA 3      | meta-llama/Llama-3-*-hf |  ✓      | ✓       | ✓                  | ✓                                   | ✓       | ✓           | ✓                      | ✓                |
+--------------+-------------------------+---------+---------+--------------------+-------------------------------------+---------+-------------+------------------------+------------------+
| OPT          | facebook/opt-*          |  ✓      | ✓       | ✓                  | ✓                                   | ✓       | ✓           | ✗                      | ✓                |
+--------------+-------------------------+---------+---------+--------------------+-------------------------------------+---------+-------------+------------------------+------------------+
| Qwen 1.5     | Qwen/Qwen1.5-*          |  ✓      | ✓       | ✓                  | ✓                                   | ✓       | ✓           | ✗                      | ✓                |
+--------------+-------------------------+---------+---------+--------------------+-------------------------------------+---------+-------------+------------------------+------------------+


Note: ``*`` represents different model sizes, such as ``7b``.

Preparation
-----------

For Llama2 models, download the HF Llama2 checkpoint. The Llama2 models checkpoint can be accessed by submitting a permission request to Meta.
For additional details, see the `Llama2 page on Huggingface <https://huggingface.co/docs/transformers/main/en/model_doc/llama2>`__. Upon obtaining permission, download the checkpoint to the ``[llama2_checkpoint_folder]``.

Quantization & Export Scripts
-----------------------------

You can run the following python scripts in the ``examples/torch/language_modeling`` path. Here we use Llama2-7b as an example.

**Recipe 1: Evaluation of Llama2 float16 model without quantization**

::

   python3 quantize_quark.py --model_dir [llama2 checkpoint folder] \
                             --skip_quantization

**Recipe 2: Quantization & Json_SafeTensors_Export of Llama2 with AWQ (W_uint4 A_float16 per_group asymmetric)**

::

   python3 quantize_quark.py --model_dir [llama2 checkpoint folder] \
                             --output_dir output_dir \
                             --quant_scheme w_uint4_per_group_asym \
                             --num_calib_data 128 \
                             --quant_algo awq \
                             --dataset pileval_for_awq_benchmark \
                             --seq_len 512 \
                             --model_export quark_safetensors

If the code runs successfully, it will produce one JSON file and one .safetensor file in ``[output_dir]`` and the terminal will display ``[Quark] Json-safetensors quantized model exported to ... successfully.``

**Recipe 3: Quantization & Json_SafeTensors_Export of Llama2 with W_int4 A_float16 per_group symmetric**

::

   python3 quantize_quark.py --model_dir [llama2 checkpoint folder] \
                             --output_dir output_dir \
                             --quant_scheme w_int4_per_group_sym \
                             --num_calib_data 128 \
                             --model_export quark_safetensors

If the code runs successfully, it will produce one JSON file and one .safetensor file in ``[output_dir]`` and the terminal will display ``[Quark] Json-safetensors quantized model exported to ... successfully.``

**Recipe 4: Quantization & Json_SafeTensors_Export of W_FP8_A_FP8 with FP8 KV cache**

::

   python3 quantize_quark.py --model_dir [llama2 checkpoint folder] \
                             --output_dir output_dir \
                             --quant_scheme w_fp8_a_fp8 \
                             --kv_cache_dtype fp8 \
                             --num_calib_data 128 \
                             --model_export quark_safetensors

If the code runs successfully, it will produce one JSON file and one .safetensor file in ``[output_dir]`` and the terminal will display ``[Quark] Json-safetensors quantized model exported to ... successfully.``

**Recipe 5: Quantization & Json_SafeTensors_Export of only W_FP8_A_FP8**

::

   python3 quantize_quark.py --model_dir [llama2 checkpoint folder] \
                             --output_dir output_dir \
                             --quant_scheme w_fp8_a_fp8 \
                             --num_calib_data 128 \
                             --model_export quark_safetensors

If the code runs successfully, it will produce one JSON file and one .safetensor file in ``[output_dir]`` and the terminal will display ``[Quark] Json-safetensors quantized model exported to ... successfully.``

**Recipe 6: Quantization & Json_SafeTensors_Export of W_FP8_A_FP8_O_FP8**

::

   python3 quantize_quark.py --model_dir [llama2 checkpoint folder] \
                             --output_dir output_dir \
                             --quant_scheme w_fp8_a_fp8_o_fp8 \
                             --num_calib_data 128 \
                             --model_export quark_safetensors

If the code runs successfully, it will produce one JSON file and one .safetensor file in ``[output_dir]`` and the terminal will display ``[Quark] Json-safetensors quantized model exported to ... successfully.``

**Recipe 7: Quantization & Json_SafeTensors_Export of W_FP8_A_FP8_O_FP8 without weight scaling factors merged.** And if option
"-no_weight_matrix_merge" is not set, weight scaling factors of are merged.

::

   python3 quantize_quark.py --model_dir [llama2 checkpoint folder] \
                             --output_dir output_dir \
                             --quant_scheme w_fp8_a_fp8_o_fp8 \
                             --num_calib_data 128 \
                             --model_export quark_safetensors \
                             --no_weight_matrix_merge

If the code runs successfully, it will produce one JSON file and one .safetensor file in ``[output_dir]`` and the terminal will display ``[Quark] Quantized model exported to ... successfully.``

**Recipe 8: Quantization & vLLM_Adopt_SafeTensors_Export of
W_FP8_A_FP8_O_FP8**

::

   python3 quantize_quark.py --model_dir [llama2 checkpoint folder] \
                             --output_dir output_dir \
                             --quant_scheme w_fp8_a_fp8_o_fp8 \
                             --num_calib_data 128 \
                             --model_export vllm_adopted_safetensors

If the code runs successfully, it will produce one JSON file and one .safetensor file in ``[output_dir]`` and the terminal will display ``[Quark] VLLM adopted quantized model exported to ... successfully.``

**Recipe 9: Quantization & Torch compile of W_INT8_A_INT8_PER_TENSOR_SYM**

::

   python3 quantize_quark.py --model_dir [llama2 checkpoint folder] \
                             --output_dir output_dir \
                             --quant_scheme w_int8_a_int8_per_tensor_sym \
                             --num_calib_data 128 \
                             --device cpu \
                             --data_type bfloat16 \
                             --model_export torch_compile

**Recipe 10: Quantization & GGUF_Export with AWQ (W_uint4 A_float16 per_group asymmetric)**

::

   python3 quantize_quark.py --model_dir [llama2 checkpoint folder] \
                             --output_dir output_dir \
                             --quant_scheme w_uint4_per_group_asym \
                             --quant_algo awq \
                             --num_calib_data 128 \
                             --group_size 32 \
                             --model_export gguf

If the code runs successfully, it will produce one gguf file in ``[output_dir]`` and the terminal will display ``GGUF quantized model exported to ... successfully.``

**Recipe 11: MX Quantization**

Quark now supports the datatype microscaling which is abbreviated as MX. Use the following command to quantize model to datatype MX:

::

   python3 quantize_quark.py --model_dir [llama2 checkpoint folder] \
                             --output_dir output_dir \
                             --quant_scheme w_mx_fp8 \
                             --num_calib_data 32 \
                             --group_size 32

The command above is weight-only quantization. If uses want activations to be quantized as well, use the command below:

::

   python3 quantize_quark.py --model_dir [llama2 checkpoint folder] \
                             --output_dir output_dir \
                             --quant_scheme w_mx_fp8_a_mx_fp8 \
                             --num_calib_data 32 \
                             --group_size 32

.. raw:: html

   <!--
   ## License
   Copyright (C) 2023, Advanced Micro Devices, Inc. All rights reserved. SPDX-License-Identifier: MIT
   -->
