# serve/

Model-serving pipeline: pull a fine-tuned checkpoint, merge the LoRA adapter into the base model, and launch a vLLM OpenAI-compatible API for downstream evaluation.

## Files

| File | Purpose |
|------|---------|
| `load_checkpoint.py` | Downloads a checkpoint from the Hugging Face Hub at a specific commit revision. |
| `load_custom_model.py` | Loads a base Gemma-4 model, merges in the downloaded LoRA adapter, and writes the merged model + tokenizer to disk so vLLM can serve it. |
| `eval_inference.py` | Batched inference client against a running vLLM server. Reads JSON prompts and writes responses. |
| `vllm.sh` | vLLM startup script. Serves the merged model with `bfloat16`, prefix caching, tool-calling enabled, and the custom Gemma-4 chat template. |
| `tool_chat_template_gemma4.jinja` | Custom Jinja2 chat template implementing Gemma-4's tool-calling format. |
| `serve/` | Duplicated copies of the load + inference scripts (alternate / legacy configuration). |

## Example

```bash
python serve/load_checkpoint.py -r Ayodeji/gemma-4-31B-present_god -c <commit-sha>
python serve/load_custom_model.py -t Ayodeji/gemma-4-31B-present_god -c <commit-sha> -m google/gemma-4-31B
bash serve/vllm.sh
```

Then point evaluators at `http://localhost:8000/v1` (use SSH port-forwarding if the VM is remote — see `compute/steps.md`).
