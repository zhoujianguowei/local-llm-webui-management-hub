# Mixed Multimodal LLM Management System

## 📋 System Overview

This system (Mixed Multimodal LLM Management System) is a comprehensive management platform for locally deployed large language models, integrating **file management, user permissions, model scheduling, AI chat, and system monitoring** into one secure, efficient, and user-friendly solution.

Built on Java + Spring Boot with Bootstrap 5 on the frontend, the system schedules local GGUF-formatted large language models through llama.cpp on the backend, while also supporting OpenAI-compatible remote API integration. Its core objective is to address cumbersome path permission management, model parameter configuration, and resource monitoring in local model deployments.

![Main Interface](imgs/base_main.png)

**Version Information**

| Version | Version Suffix | Description | Included Features |
| :--- | :--- | :--- | :--- |
| **Base Edition** | `release_base` | Minimal deployment version | Core features: file management, user management, AI chat, permission system |
| **Ultimate Edition** | `release_ultimate` | Full-featured version | All base features + model management, resource monitoring, scheduled shutdown/reboot, and more |

> ⚠️ **Multimodal Capability Note**: The current version's multimodal capabilities **only support image input**; audio and video input are not supported.
>
> ⚠️ **Scheduled Shutdown/Reboot Compatibility Note**: This feature relies on underlying operating system commands and hardware support; not all machines can run it properly. Tested systems: dual X99 (Ubuntu 22.04, E5-2696 v4) and Mac M1 Pro shutdown/reboot work normally; Windows 10 64-bit (z690 motherboard, i7-13700K) shutdown works, but automatic wake-up reboot is limited by motherboard BIOS and cannot be woken up in self-testing.
>
> ⚠️ **GPU Monitoring Compatibility Note**: GPU monitoring depends on the `nvidia-smi` tool and **only supports systems with NVIDIA GPUs**. macOS systems using Apple Silicon (M series) or integrated graphics have no corresponding monitoring interface, so no GPU monitoring panel is provided under macOS.



## 🌟 Core Feature Highlights

### 1. Minimal Deployment and Multi-Platform Compatibility

- **Zero Database Dependency**: No need to install MySQL/PostgreSQL or other databases; single JAR deployment, runs on startup.
- **Multi-Platform Support**: Compatible with **Windows**, **Linux (Ubuntu)**, and **macOS (Apple Silicon)**.
- **Fully Offline Operation**: Frontend resources are 100% localized, with no dependency on external CDNs, meeting high-security intranet requirements.

### 2. File Management

- **Smart Upload**: Supports drag-and-drop file upload and chunked upload for large files, with real-time progress display for each chunk and overall percentage in the right task bar.

![File Upload Progress](imgs/file-upload-progress.png)

- **Flexible Download**: Supports file or folder download; individual files download directly, while folders are automatically packaged into `.zip` archives in the background, with completion status viewable in the download task list.
- **Multimedia Preview**: Supports preview of multiple multimedia formats without download.
  - Images (JPG, PNG): Full-screen view support
  - Video: Range request support, freely draggable progress bar
  - Documents (PDF): Online preview support
  - Text/Code: Preview support for common text formats including TXT, Java, Python, JS, HTML, JSON, YAML, etc.
- **Search**: Supports fuzzy filename matching with wildcard support (e.g., `*.py`) for quick filtering of specific file types.
- **View Switching**: Toggle between list view (detailed properties) and tile view (suitable for image browsing).
- **Secure Deletion**: Deletion requires double confirmation, strict `DELETE` permission validation, and protected paths cannot be deleted even by administrators.

### 3. Fine-Grained Permission System

![Permission Tree Management](imgs/permission-tree.png)

- **Path Inheritance Mechanism**: When accessing files without set rules, the system automatically traces upward to the parent directory until the nearest matching rule is found.

![Permission Edit](imgs/permission-edit.png)

- **Permission Cascade (Priority)**: **Delete > Upload > Download > Execute > Read**; higher-level permissions automatically include lower-level permissions.
- **Dual Role and User Validation**: Each permission can configure minimum allowed role, with support for specific authorized user lists.
- **Path Penetration**: Even without parent directory execution permissions, files with read permissions at deeper levels can be accessed directly.
- **Permission Tree Management**: Visual tree structure for intuitive viewing of system-wide permission distribution, with quick location and modification of specific subdirectory permissions.
- **Path Protection Mechanism**: Protected paths display special identifiers in the file management interface, with all deletion requests directly intercepted.

### 4. Full User Lifecycle Management

![User Management](imgs/user-list.png)

- **Role Management**: Supports three roles — Admin, User, and Guest — switchable at any time.
- **Status Control**: One-click ban/unban; banned users trigger an unban request process upon login, which administrators can handle in real time.
- **Guest Expiry**: Configurable default validity period (in days) for guest accounts; the system periodically scans for expired accounts and automatically bans them.
- **Self-Registration**: When guest registration is enabled, users can self-register on the login page, with accounts automatically receiving preset validity periods.

### 5. Intelligent AI Chat

![AI Chat](imgs/chat-main.png)

- **Conversation Management**: Supports new conversations, loading history, deleting sessions, renaming, and pinning frequently used conversations.
- **Multi-Method Attachment Upload**:
  - Image Upload: AI can recognize text content within images
  - Text File Upload: Supports common text types including `.txt`, `.java`, `.py`, `.js`, `.bat`, `.sh`, etc.

![Multi-Method Attachment Upload](imgs/multi-input.png)

  - Folder Upload: Supports uploading entire directory listings. If the model supports image input, image files are included; otherwise, only plain text files are included.

![Project-Level Code Analysis](imgs/chat-project-analysis.png)

  - Paste File Path: Paste paths into the input box for automatic recognition and upload
  - Copy File Paste: Copy files from a file manager or IDE and paste directly into the chat input box

- **Code Generation**: AI-generated code blocks support one-click copy, save as local file, and collapsible display for long code.
- **Deep Thinking Mode**: Supports deep thinking functionality for models such as qwen3.5, qwen3.6, gemma4, deepseek, etc. A highlighted lightbulb icon in the input box indicates the feature is enabled.
- **System-Level Configuration**: Administrators can configure OpenAPI integration, model capability definitions (regex matching to enable/disable thinking mode), and per-role limits on attachment size and maximum message count.
- **Conversation Statistics**: Real-time display of prompt prefill speed, decode speed, current conversation context ratio, and other performance metrics.

### 6. Deep Integration with llama.cpp Inference Engine
 **Also supports adding ik_llama.cpp as an inference engine (optional)**. If this path is configured, model startup supports selecting between llama.cpp and ik_llama.cpp inference engines. (GGUF auto-sharding still uses llama.cpp at the底层; for ik_llama.cpp sharding, manual merging may be required.)

![Model List](imgs/model-list-overview.png)

- **Multi-GPU Parallel Acceleration**: Full support for llama.cpp's **SM Tensor** scheduling for multi-GPU parallel inference.
- **MTP Prediction Acceleration**: Supports **Multi-Token Prediction** to significantly reduce inference steps.
- **Multiple Quantization Format Support**: Supports F32/BF16/F16Q8_0/Q8_K, Q6_K, Q5_K_M, Q4_K_M, IQ3_XXS, and other common quantization types.
- **GGUF Automated Management**:
  - Automatic detection of sharded files (e.g., `model-00001-of-00002.gguf`) with one-click merge
  - Built-in naming convention checks to ensure uniform model file naming (format: `model_name_quantization_type.gguf`)
  - Automatic matching of `mmproj` multimodal projection files and `mtp` Draft model files
- **Fine-Grained Startup Parameters**: Supports 18+ parameter tuning including context size, GPU load layers, parallel task count, tensor splitting, KV Cache quantization, temperature, thread count, batch size, and more.

![GPU Configuration](imgs/model-gpu-config.png)

### 7. Real-Time Resource Monitoring

![GPU Monitoring](imgs/model-gpu-overview.png)

- **GPU Monitoring**: Integrated `nvidia-smi` real-time parsing, data synchronized every 10 seconds, recording trend charts of the last 60 samples for intuitive viewing of each GPU's type and memory usage.

![Memory Monitoring](imgs/model-mem.png)

- **Memory Monitoring**: Real-time display of total memory, used memory, available memory, and overall usage percentage, with a 3-minute memory usage fluctuation curve to help analyze memory peaks during model loading.

### 8. Intelligent Scheduled Reboot

![Scheduled Reboot](imgs/reboot-add-detailed.png)

- **Dual-Mode Tasks**:
  - One-Time Tasks: Automatically deleted after execution, suitable for temporary maintenance and single experiments
  - Periodic Tasks: Configurable workdays with automatic next-cycle calculation upon expiry, suitable for daily scheduled shutdown/wake
- **Task Management**: View all created tasks including execution time, status (pending/running/expired), and real-time countdown, with support for deletion or delay.
- **Expiry Warning**: When a task is less than 10 minutes from execution, a countdown warning card pops up at the top of the page.

![Reboot Warning](imgs/reboot-warning.png)

---

## 👥 User Role Description

| Role | Permission Level | Description | Accessible Features |
| :--- | :--- | :--- | :--- |
| **Admin** | Highest (Level 3) | System Maintainer | All features: file management, user management, model management, resource monitoring, scheduled shutdown, global configuration |
| **User** | Standard (Level 2) | Registered User | File management (permission-restricted), AI chat, personal settings; accounts have expiry dates with automatic ban upon expiry |
| **Guest** | Temporary (Level 1) | Self-Registered Account | File management (permission-restricted), AI chat, personal settings; accounts have expiry dates with automatic ban upon expiry |


---

## 🔒 Offline Authorization and Physical Binding

The Ultimate Edition uses a **one-machine-one-code offline activation** mechanism. The system runs 100% offline during operation, requiring no external network communication. Core code is protected with high-intensity security obfuscation and encryption, ensuring absolute data privacy in enterprise and personal intranet environments.

The system's machine code is determined by your **CPU, motherboard, and operating system**:
* **Worry-Free Hardware Upgrades**: Routine upgrades of GPU, hard drive, RAM, or power supply **do not affect** the system's activation status.
* **Reset Triggers**: Machine code changes only occur when reinstalling the underlying operating system or replacing core components (CPU/motherboard).
