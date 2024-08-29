# TextGenTips
Collection of tips for using textgen in various ways

**Single GPU TIPS**
A collection of resources to help folks get a lot out of a single 24GB GPU (these instructions are for Nvidia cards)

Even though models are offloaded, they use a little bit of vram.  So if your favorite model uses your gpu comptelty it may not work, you may need to quantize the model or move whisper to cpu mode.

This is in reference to this reddit post: 

https://old.reddit.com/r/LocalLLaMA/comments/1e3aboz/folks_with_one_24gb_gpu_you_can_use_an_llm_sdxl/
https://www.reddit.com/r/LocalLLaMA/comments/1e3aboz/folks_with_one_24gb_gpu_you_can_use_an_llm_sdxl/

It's important to familiarize yourself with each extension on its own before trying to use them all at once, and to watch your GPU VRAM to make sure things are working as expected.

This is the order of my CMD_Flag, the order is important, if you don't want to use an extension remove it form the list but don't change the order (If you are clicking the UI radio buttons to load your extensions, you need to click each button in the same sequence as the --extensions flag shows

`--extensions text-generation-webui-model_ducking Lucid_Vision alltalk_tts whisper_stt sd_api_pictures`

1. Get text-generation-webui here: https://github.com/oobabooga/text-generation-webui/releases

2. After installation, install the text-generation-webui-model_ducking extension here: https://github.com/oobabooga/text-generation-webui-extensions?tab=readme-ov-file#model-ducking

3. Load a model in textgen and test out the model_ducking extension, make sure it is working for you.

4. Install Lucid_Vision here: https://github.com/oobabooga/text-generation-webui-extensions?tab=readme-ov-file#lucid_vision

5. If you do install Lucid_Vision do a test image in the UI without involving the LLM, I don't know what the issue is, but often gradio will timeout the first time you load a model (sometimes LLM models) when this happens the vision model received nothing and the ui will say "error" just try another picture and things should work well from that point on (this is mentioned in the Lucid_Vision repo as well)

6. I recommend disabling DeepseekVL when you first use Lucid_Vision https://github.com/RandomInternetPreson/Lucid_Vision?tab=readme-ov-file#model-information  The reason being is that to use it you need to import their dependencies, which may cause issues.  If things are working for you and you want to use deepseekVL feel free to install it as per the instructions it's unlikely to cause conflicts but not guaranteed.

7. Lucid_Vision relies on your LLMs abilities to contextualize how to use the extension by itself, the manual image upload will work regardless of your LLM.

8. Install alltalk_tts here: https://github.com/oobabooga/text-generation-webui-extensions?tab=readme-ov-file#alltalk-tts

9. erew123 Has a lot of good documentation on how to get Alltalk running in windows and linux, if you want to get the render speeds of the video you need to install deepspeed.  It works in Linux just fine, but Windows needs to install the prebuilt deepseed wheels.  https://github.com/erew123/alltalk_tts?#-deepspeed-installation-options

10. To install deepspeed this is how I do it (these are not instructions, there are other steps in erew123's documentation, but every new install only nees roughly these steps), read erew123's documentation depending on your system (windows is a bit easier):

```
*via the cmd_linux.sh file depending on your os, then close terminal*
pip install libaio 

*Start Textgen* 

*Open Textgen Enviroment via the cmd_linux.sh file depending on your os, I have cuda 12.1 installed*
export CUDA_HOME=/usr/local/cuda-12.1

cd extensions/alltalk_tts

chmod +x atsetup.sh

./atsetup.sh

*follow installation instructions and instructions on how to install deepspeed*
```

11. If you want to use alltalk on linux this is a solution to a perimeninant export CUDA_HOME="/usr/local/cuda" if you are having issues
    https://github.com/erew123/alltalk_tts/issues/107

12. Whisper_STT comes with textgen you just need to install the dependendies, oobabooga has created an update_wizard file for each operating system, simply click on that and follow the instructions to install the requried Whisper_STT dependencies

13. Okay now the last extension, sd_api_pictures.  You don't need to install anything for this extension, but if you want the same functionality as the video, you need to replace your script.py file in the sd_api_pictures extensions folder with this version: https://raw.githubusercontent.com/RandomInternetPreson/TextGenTips/main/sd_api_pictures/with_ADetailer/script.py
    
This is a simplified version of the original version with some changes that unload and load the model from GPU RAM to and from CPU RAM, auto1111's api will be used when the words "send" and "prompt" are see in the users' message to the LLM.  So if you type "send me a prompt of a kitten attacking a great white shark underwater" the LLMs response to you will be fed to the stable diffusion model.

This code creates images that are 1024x1024, so if you are streaming this on a mobile device it is best to use "desktop mode" else the pictures might be too big for the UI.

It is these two functions that are the main difference:

```
def load_sd_model():
    print("Loading the Stable Diffusion model into VRAM...")
    response = requests.post(url='http://127.0.0.1:7861/sdapi/v1/reload-checkpoint', json='')
    response.raise_for_status()
    del response

def unload_sd_model():
    print("Unloading the Stable Diffusion model from VRAM...")
    response = requests.post(url='http://127.0.0.1:7861/sdapi/v1/unload-checkpoint', json='')
    response.raise_for_status()
    del response
```

The original code has something like this too, but I could not get it to work with Text-generation-webui-model_ducking.

Additionally, the code uses ADetalier, so if you don't use that extension in auto1111 use this version of the code: 
https://raw.githubusercontent.com/RandomInternetPreson/TextGenTips/main/sd_api_pictures/without_ADetailer/script.py

You need to edit your webui-user in the auto1111 install directory so that Auto1111 is running on port 7861, or you can edit the script.py file


Now for my slighlty janky implementation, you need to start Auto1111 first, then open the UI and manually load the model onto CPU Ram by checking the "Uload the SD Checkpoiint to RAM" button.  You can close the auto1111 UI at this point, and don't need to restart it if you need to restart textgen.  I'll work on the code doing this automatically later.

Go to setting, then Actions at the bottom

![image](https://github.com/user-attachments/assets/0f1cfece-6410-4246-89b0-63a2cdbc4663)

Then scroll up and click the "Uload the SD Checkpoiint to RAM" button

![image](https://github.com/user-attachments/assets/87ce6127-d7a7-4f8e-af82-0df8fc4656e9)

Now start textgen and you should be good to go.

14. Test out this sd_api_pictures extension, make sure it is working for you.  You need to be clever about how you explain to your model how the extension works, usually I edit the first attmept of the LLM if it's not the right formatting and then it follows that example for subsequent prompt requests.

You can try telling your llm this:

```
I want you to send me a prompt of a starfish in a tuxedo. And so in doing that, just respond back to me with exactly this phrase, "starfish wearing tuxedo." Do not preface or add any additional text to your reponse back to me.
```



**ExllamaV2 tensor parallelism for OOB V1.14**

1. Install the updated exllamav2 repo using the textgen cmd terminal for you os (linux for example is cmd_linux.sh)
   
Within the terminal navigate to the repositories folder "cd repositories"

Run these commands 
```
git clone https://github.com/turboderp/exllamav2
cd exllamav2
pip install -r requirements.txt
pip install .
```
2. After you've done this replace your exllamav2.py file and shared.py (located in the modules folder) file with these: https://github.com/RandomInternetPreson/TextGenTips/tree/main/ExllamaV2_TensorParallel_Files
3. When running textgen add
```
--enable_tp
```
to your CMD_FLAGS.txt file
