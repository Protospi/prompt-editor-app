<template>
  <div class="row q-col-gutter-md">
    <!-- Editor Column (Left) -->
    <div class="col-6">
      <q-card class="editor-card q-mt-md shadow-5">
        <!-- Simple formatting toolbar -->
        <q-card-section class="formatting-toolbar q-py-sm bg-grey-2">
          <div class="row items-center justify-between">
            <!-- Left side formatting tools -->
            <div class="row items-center">
              <q-btn flat dense size="md" icon="format_bold" @click="applyFormat('bold')" class="title-button q-mr-xs" />
              <q-btn flat dense size="md" icon="format_list_bulleted" @click="applyFormat('insertUnorderedList')" class="title-button q-mr-xs" />
              
              <!-- Color dropdown -->
              <q-btn-dropdown 
                flat 
                dense 
                size="md" 
                icon="format_color_text" 
                class="title-button q-mr-xs"
                @show="saveSelection"
              >
                <q-list>
                  <q-item clickable v-close-popup @click="() => applyColor('#6467F2')">
                    <q-item-section>
                      <div class="row items-center">
                        <div class="color-swatch" style="background-color: #6467F2"></div>
                        <div class="q-ml-sm">Purple</div>
                      </div>
                    </q-item-section>
                  </q-item>
                  <q-item clickable v-close-popup @click="() => applyColor('#000000')">
                    <q-item-section>
                      <div class="row items-center">
                        <div class="color-swatch" style="background-color: #000000"></div>
                        <div class="q-ml-sm">Black</div>
                      </div>
                    </q-item-section>
                  </q-item>
                  <q-item clickable v-close-popup @click="() => applyColor('#FF0000')">
                    <q-item-section>
                      <div class="row items-center">
                        <div class="color-swatch" style="background-color: #FF0000"></div>
                        <div class="q-ml-sm">Red</div>
                      </div>
                    </q-item-section>
                  </q-item>
                  <q-item clickable v-close-popup @click="() => applyColor('#008000')">
                    <q-item-section>
                      <div class="row items-center">
                        <div class="color-swatch" style="background-color: #008000"></div>
                        <div class="q-ml-sm">Green</div>
                      </div>
                    </q-item-section>
                  </q-item>
                  <q-item clickable v-close-popup @click="() => applyColor('#FFA500')">
                    <q-item-section>
                      <div class="row items-center">
                        <div class="color-swatch" style="background-color: #FFA500"></div>
                        <div class="q-ml-sm">Orange</div>
                      </div>
                    </q-item-section>
                  </q-item>
                </q-list>
              </q-btn-dropdown>
              
              <!-- Title button -->
              <q-btn 
                flat 
                dense
                no-caps
                label="T"
                size="lg"
                class="title-button q-mr-xs"
                @click="formatAsTitle"
                >
                <q-tooltip>Format line as title</q-tooltip>
              </q-btn>

              <q-separator vertical inset class="q-mx-sm" />
              
              <!-- New Prompt button -->
              <q-btn 
                flat 
                dense
                icon="add"
                size="md"
                class="title-button q-mr-xs"
                @click="resetEditor"
              >
                <q-tooltip>Reset prompt</q-tooltip>
              </q-btn>

              <!-- AI button -->
              <q-btn 
                flat 
                dense
                :icon="aiModeActive ? 'edit' : 'auto_awesome'"
                size="md"
                :color="aiModeActive ? 'positive' : '#6467F2'"
                class="title-button q-mr-xs"  
                @click="toggleAiMode"
              >
                <q-tooltip>{{ aiModeActive ? 'Return to editor' : 'Use AI assistant to help with your prompt' }}</q-tooltip>
              </q-btn>

            </div>
            
            <!-- Right side version controls -->
            <div class="row items-center">
              <!-- Version selector dropdown -->
              <q-btn-dropdown 
                flat 
                dense 
                no-caps
                :label="currentVersionLabel"
                class="version-selector q-mr-sm"
                color="primary"
                icon="history"
              >
                <q-list>
                  <template v-for="(version, index) in promptVersions" :key="index">
                    <q-item v-if="version" clickable @click="loadVersion(index)">
                      <q-item-section>
                        <div class="row items-center">
                          <q-icon 
                            :name="index === currentVersionIndex ? 'bookmark' : 'bookmark_border'" 
                            size="xs" 
                            class="q-mr-xs" 
                            :color="index === currentVersionIndex ? 'primary' : 'grey-7'" 
                          />
                          <div>{{ version.name }}</div>
                        </div>
                      </q-item-section>
                      <q-item-section side class="row no-wrap">
                        <q-btn flat round dense icon="edit" size="xs" @click.stop="promptForRename(index)">
                          <q-tooltip>Rename</q-tooltip>
                        </q-btn>
                        <q-btn flat round dense icon="delete" size="xs" color="negative" @click.stop="promptForDelete(index)">
                          <q-tooltip>Delete version</q-tooltip>
                        </q-btn>
                      </q-item-section>
                    </q-item>
                  </template>
                </q-list>
              </q-btn-dropdown>
              
              <!-- Save version button -->
              <q-btn 
                flat 
                dense 
                :color="hasUnsavedChanges && !allSlotsFilled ? 'positive' : 'grey-6'" 
                icon="save" 
                @click="saveVersion"
                class="save-version-btn"
                :disable="!hasUnsavedChanges || allSlotsFilled"
              >
                <q-tooltip>{{ getSaveButtonTooltip() }}</q-tooltip>
              </q-btn>
            </div>
          </div>
        </q-card-section>
        
        <!-- Editor area -->
        <q-card-section class="editor-content">
          <!-- Regular editor when AI mode is off -->
          <div 
            v-if="!aiModeActive"
            ref="editorEl" 
            class="editor" 
            contenteditable="true" 
            @input="updateMarkdown"
            @keydown="handleKeyDown"
            @keyup="updateMarkdown"
          ></div>
          
          <!-- AI editor mode when AI mode is on -->
          <div v-else class="ai-editor-container">
            <div class="text-subtitle1 q-mb-md">
              <q-icon name="smart_toy" class="q-mr-xs" /> AI Prompt Assistant
            </div>
            
            <p class="text-body2 q-mb-sm">Describe the prompt you'd like to create or edit:</p>
            <p class="text-caption text-grey-8 q-mb-md">
              Example: "I need a prompt that helps a language model act as a coding tutor for beginners" 
              or "Create a prompt for a storytelling assistant that creates children's stories"
            </p>
            
            <q-input
              v-model="aiPromptInput"
              type="textarea"
              outlined
              rows="12"
              label="Your instructions"
              placeholder="Describe what you want, and the AI will help create or refine your prompt..."
              :disable="isProcessingAiRequest"
              class="q-mb-md"
              autofocus
            />
            
            <!-- Action buttons at the bottom -->
            <div class="row justify-end q-mt-md">
              <q-btn 
                label="Back to Editor" 
                color="grey-7" 
                @click="toggleAiMode"
                :disable="isProcessingAiRequest" 
                flat
                class="q-mr-sm"
                icon="arrow_back"
              >
                <q-tooltip>Return to the regular editor without changes</q-tooltip>
              </q-btn>
              <q-btn 
                label="Generate Suggestion" 
                color="primary" 
                :loading="isProcessingAiRequest"
                :disable="!aiPromptInput.trim()"
                @click="submitAiPrompt"
                icon-right="auto_awesome"
              >
                <template v-slot:loading>
                  <q-spinner-dots />
                </template>
                <q-tooltip>Generate AI suggestions based on your instructions</q-tooltip>
              </q-btn>
            </div>
          </div>
        </q-card-section>
      </q-card>
    </div>
    
    <!-- Markdown Output Column (Right) -->
    <div class="col-6">
      <!-- Markdown Output Card -->
      <q-card class="markdown-card q-mt-md shadow-3">
        <q-card-section class="markdown-header">
          <div class="text-subtitle1 text-weight-medium" style="color: #6467F2;">
            <q-icon name="code" class="q-mr-sm" />
            Markdown Output
          </div>
          <div class="text-caption text-grey-7">
            This is the markdown that will be sent to the AI model
          </div>
        </q-card-section>
        
        <q-separator />
        
        <q-card-section class="markdown-content">
          <q-input
            type="textarea"
            v-model="markdownOutput"
            readonly
            filled
            autogrow
            class="markdown-output"
            bg-color="#f5f5f5"
          />
        </q-card-section>
      </q-card>
    </div>
    
    <!-- Dialog for renaming versions -->
    <q-dialog v-model="renameDialog.show" persistent>
      <q-card style="min-width: 350px">
        <q-card-section>
          <div class="text-h6">Rename Version</div>
        </q-card-section>
        
        <q-card-section>
          <q-input v-model="renameDialog.newName" label="Version name" autofocus />
        </q-card-section>
        
        <q-card-actions align="right">
          <q-btn flat label="Cancel" color="negative" v-close-popup />
          <q-btn flat label="Save" color="positive" @click="renameVersion" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>
    
    <!-- New Prompt Confirmation Dialog -->
    <q-dialog v-model="newPromptDialog.show" persistent>
      <q-card>
        <q-card-section class="row items-center">
          <q-avatar icon="warning" color="warning" text-color="white" />
          <span class="q-ml-sm">You have unsaved changes. Creating a new prompt will discard these changes.</span>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="Cancel" color="primary" v-close-popup />
          <q-btn flat label="Discard Changes" color="negative" @click="confirmNewPrompt" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>
    
    <!-- Delete Confirmation Dialog -->
    <q-dialog v-model="deleteDialog.show" persistent>
      <q-card>
        <q-card-section class="row items-center">
          <q-avatar icon="warning" color="negative" text-color="white" />
          <span class="q-ml-sm">
            Are you sure you want to delete "{{ deleteDialog.versionName }}"?
            <span v-if="deleteDialog.isCurrentVersion" class="text-negative">
              <br>This will reset the editor since it's the currently displayed version.
            </span>
          </span>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="Cancel" color="primary" v-close-popup />
          <q-btn flat label="Delete" color="negative" @click="confirmDelete" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';
import turndown from 'turndown';
import { useQuasar } from 'quasar';

// Initialize Quasar
const $q = useQuasar();

// Types for the version system
interface PromptVersion {
  name: string;
  timestamp: string;
  content: string;
  html: string;
}

// Initialize the Turndown service for HTML to Markdown conversion
const turndownService = new turndown({
  headingStyle: 'atx',
  codeBlockStyle: 'fenced'
});

// Editor element reference
const editorEl = ref<HTMLElement | null>(null);

// Markdown output
const markdownOutput = ref('');

// Store text selection for color change
const savedSelection = ref<Range | null>(null);

// AI Mode state
const aiModeActive = ref(false);
const aiPromptInput = ref('');
const isProcessingAiRequest = ref(false);

// Version management
const promptVersions = ref<(PromptVersion | null)[]>([null, null, null, null, null]);
const currentVersionIndex = ref<number | null>(null);
const lastSavedContent = ref<string>('');
const hasUnsavedChanges = computed(() => {
  // If no version is currently selected
  if (currentVersionIndex.value === null) {
    // Check if the content is different from the sample text
    const isNotInitialContent = markdownOutput.value !== turndownService.turndown('<p>Comece a escrever seu prompt aqui...</p>');
    // Check if it's different from the last saved content (handles AI suggestions case)
    const isDifferentFromLastSaved = markdownOutput.value !== lastSavedContent.value;
    
    return isNotInitialContent || isDifferentFromLastSaved;
  }
  
  // Otherwise, check if content has changed since last save
  return markdownOutput.value !== lastSavedContent.value;
});

// Dialogs
const renameDialog = ref({
  show: false,
  index: 0,
  newName: ''
});

const newPromptDialog = ref({
  show: false
});

const deleteDialog = ref({
  show: false,
  index: 0,
  versionName: '',
  isCurrentVersion: false
});

// Computed properties
const currentVersionLabel = computed(() => {
  if (currentVersionIndex.value !== null && promptVersions.value[currentVersionIndex.value]) {
    return promptVersions.value[currentVersionIndex.value]?.name || 'Current Version';
  }
  return 'Unsaved Version';
});

// Add computed property for slot status
const allSlotsFilled = computed(() => {
  return !promptVersions.value.some(v => v === null);
});

// Get tooltip message for save button based on state
const getSaveButtonTooltip = () => {
  if (!hasUnsavedChanges.value) {
    return 'Make changes to enable saving';
  }
  
  if (allSlotsFilled.value) {
    return 'Max 5 prompts reached. Delete a version to save new changes.';
  }
  
  return 'Save current prompt as a version';
};

// Apply basic formatting
const applyFormat = (command: string) => {
  document.execCommand(command, false);
  updateMarkdown();
};

// Save selection before dropdown opens
const saveSelection = () => {
  const selection = window.getSelection();
  if (selection && selection.rangeCount > 0) {
    savedSelection.value = selection.getRangeAt(0).cloneRange();
  }
};

// Apply text color
const applyColor = (colorValue: string) => {
  // If we have a saved selection, restore it
  if (savedSelection.value) {
    const selection = window.getSelection();
    if (selection) {
      selection.removeAllRanges();
      selection.addRange(savedSelection.value);
      document.execCommand('foreColor', false, colorValue);
      savedSelection.value = null;
      updateMarkdown();
    }
  } else {
    // Fallback if no selection was saved
    document.execCommand('foreColor', false, colorValue);
    updateMarkdown();
  }
};

// Format as title (H6)
const formatAsTitle = () => {
  // Get the current selection
  const selection = window.getSelection();
  if (!selection || !selection.anchorNode) return;
  
  // Find the current line/paragraph element
  let currentNode: Node | null = selection.anchorNode;
  
  // If we're in a text node, get its parent
  if (currentNode.nodeType === Node.TEXT_NODE) {
    currentNode = currentNode.parentNode;
  }
  
  // Handle toggle behavior - if already a heading, convert to paragraph
  if (currentNode && currentNode.nodeName === 'H6') {
    document.execCommand('formatBlock', false, 'p');
  } else {
    // Find the paragraph or parent element that contains this node
    while (currentNode && currentNode.nodeName !== 'P' && currentNode.nodeName !== 'DIV') {
      currentNode = currentNode.parentNode;
    }
    
    // Apply heading format only to this line
    if (currentNode) {
      // Save selection
      const range = selection.getRangeAt(0);
      
      // Select entire paragraph
      const newRange = document.createRange();
      newRange.selectNodeContents(currentNode);
      selection.removeAllRanges();
      selection.addRange(newRange);
      
      // Apply heading format
      document.execCommand('formatBlock', false, 'h6');
      
      // Find the newly created h6 element to apply inline styles
      const heading = selection.anchorNode;
      if (heading && heading.nodeName === 'H6') {
        // Apply custom styles directly to the element for very tight spacing
        const headingEl = heading as HTMLElement;
        
        // Apply very aggressive spacing reduction
        headingEl.style.marginTop = '0';
        headingEl.style.marginBottom = '0';
        headingEl.style.paddingTop = '0';
        headingEl.style.paddingBottom = '0';
        headingEl.style.lineHeight = '1.1';
        
        // Also adjust spacing of previous and next sibling paragraphs if they exist
        if (headingEl.previousElementSibling && headingEl.previousElementSibling.tagName === 'P') {
          const prevP = headingEl.previousElementSibling as HTMLElement;
          prevP.style.marginBottom = '0';
          prevP.style.paddingBottom = '0';
        }
        
        if (headingEl.nextElementSibling && headingEl.nextElementSibling.tagName === 'P') {
          const nextP = headingEl.nextElementSibling as HTMLElement;
          nextP.style.marginTop = '0';
          nextP.style.paddingTop = '0';
        }
      }
      
      // Restore original selection
      selection.removeAllRanges();
      selection.addRange(range);
    }
  }
  
  // Add a small delay before updating markdown to ensure DOM changes are processed
  setTimeout(() => {
    updateMarkdown();
  }, 0);
};

// Handle keydown events for tabs in lists
const handleKeyDown = (event: KeyboardEvent) => {
  // Tab in lists for indentation
  if (event.key === 'Tab' && isInList()) {
    event.preventDefault();
    
    if (event.shiftKey) {
      document.execCommand('outdent', false);
    } else {
      document.execCommand('indent', false);
    }
    
    updateMarkdown();
  }
};

// Helper to check if cursor is in a list
const isInList = (): boolean => {
  const selection = window.getSelection();
  if (!selection || !selection.anchorNode) return false;
  
  let currentNode: Node | null = selection.anchorNode;
  
  while (currentNode) {
    if (currentNode.nodeName === 'LI') return true;
    currentNode = currentNode.parentNode;
  }
  
  return false;
};

// Update markdown output
const updateMarkdown = () => {
  if (editorEl.value) {
    const html = editorEl.value.innerHTML;
    markdownOutput.value = turndownService.turndown(html);
  }
};

// Version management functions
const createTimestamp = (): string => {
  const now = new Date();
  const date = now.toLocaleDateString();
  const time = now.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
  return `${date} - ${time}`;
};

const getDefaultVersionName = (index: number): string => {
  return `V${index + 1} ( ${createTimestamp()} )`;
};

const saveVersion = () => {
  // Don't allow saving if there are no changes or all slots are filled
  if (!hasUnsavedChanges.value || allSlotsFilled.value) {
    return;
  }
  
  // Check if all slots are filled - this is redundant now but kept for safety
  const emptySlotIndex = promptVersions.value.findIndex(v => v === null);
  
  if (emptySlotIndex !== -1) {
    // We have an empty slot, save to it
    createNewVersion(emptySlotIndex);
  } else {
    // This should not be reached due to the disable check, but kept for safety
    $q.notify({
      color: 'warning',
      message: 'All slots are filled. Please delete a version first.',
      icon: 'warning',
      timeout: 3000
    });
  }
};

const createNewVersion = (index: number) => {
  if (!editorEl.value) return;
  
  const version: PromptVersion = {
    name: getDefaultVersionName(index),
    timestamp: createTimestamp(),
    content: markdownOutput.value,
    html: editorEl.value.innerHTML
  };
  
  promptVersions.value[index] = version;
  currentVersionIndex.value = index;
  lastSavedContent.value = markdownOutput.value;
  
  $q.notify({
    color: 'positive',
    message: `Saved as ${version.name}`,
    icon: 'save'
  });
};

const loadVersion = (index: number) => {
  const version = promptVersions.value[index];
  if (!version || !editorEl.value) return;
  
  editorEl.value.innerHTML = version.html;
  updateMarkdown();
  currentVersionIndex.value = index;
  lastSavedContent.value = version.content;
  
  $q.notify({
    color: 'info',
    message: `Loaded ${version.name}`,
    icon: 'history'
  });
};

const promptForRename = (index: number) => {
  const version = promptVersions.value[index];
  if (!version) return;
  
  renameDialog.value.index = index;
  renameDialog.value.newName = version.name;
  renameDialog.value.show = true;
};

const renameVersion = () => {
  const { index, newName } = renameDialog.value;
  const version = promptVersions.value[index];
  
  if (version && newName.trim()) {
    version.name = newName.trim();
    $q.notify({
      color: 'positive',
      message: 'Version renamed',
      icon: 'edit'
    });
  }
};

const promptForDelete = (index: number) => {
  const version = promptVersions.value[index];
  if (!version) return;
  
  deleteDialog.value.index = index;
  deleteDialog.value.versionName = version.name;
  deleteDialog.value.isCurrentVersion = index === currentVersionIndex.value;
  deleteDialog.value.show = true;
};

const confirmDelete = () => {
  const index = deleteDialog.value.index;
  const isCurrentVersion = deleteDialog.value.isCurrentVersion;
  
  // Clear the version
  promptVersions.value[index] = null;
  
  // If this is the current version, reset the editor
  if (isCurrentVersion) {
    performReset();
  }
  
  $q.notify({
    color: 'negative',
    message: `Deleted ${deleteDialog.value.versionName}`,
    icon: 'delete',
    timeout: 2000
  });
};

// Toggle AI mode
const toggleAiMode = () => {
  // Toggle the mode
  aiModeActive.value = !aiModeActive.value;
  
  if (aiModeActive.value) {
    // Entering AI mode - reset AI input
    aiPromptInput.value = '';
  } else {
    // Exiting AI mode without generating content
    // Make sure the editor is visible and properly initialized
    // This happens if the user cancels without generating content
    setTimeout(() => {
      // Add a small delay to ensure the DOM has updated
      if (editorEl.value) {
        // Force re-render of the editor if needed
        const currentHtml = editorEl.value.innerHTML;
        if (!currentHtml || currentHtml.trim() === '') {
          editorEl.value.innerHTML = '<p>Comece a escrever seu prompt aqui...</p>';
          updateMarkdown();
        }
      }
    }, 0);
  }
};

// Submit AI prompt request
const submitAiPrompt = () => {
  if (!aiPromptInput.value.trim()) {
    $q.notify({
      color: 'negative',
      message: 'Please enter instructions for the AI',
      icon: 'warning'
    });
    return;
  }
  
  isProcessingAiRequest.value = true;
  
  // This is a temporary mock function that will be replaced with an actual API call
  setTimeout(() => {
    // Mock response with a formatted prompt template
    const mockResponse = {
      prompt: "######Persona \n Template for persona \n ######Instructions: "
    };
    
    // First exit AI mode
    aiModeActive.value = false;
    
    // Then apply the response to the editor (with a small delay to ensure DOM is ready)
    applyAiGeneratedPrompt(mockResponse.prompt);
    
    // Reset the AI processing state
    isProcessingAiRequest.value = false;
    
    $q.notify({
      color: 'positive',
      message: 'AI suggestion applied to editor. Review and save as a version if you like it.',
      icon: 'auto_awesome',
      timeout: 3000,
      position: 'top'
    });
  }, 1500); // Simulate a network delay
};

// Apply the AI generated prompt to the editor
const applyAiGeneratedPrompt = (markdownText: string) => {
  // Convert markdown to HTML
  // For a simple implementation, we'll handle the basic markdown formatting patterns
  // This would ideally use a more robust markdown-to-html converter
  const htmlContent = markdownText
    // Convert headers
    .replace(/#{6} ?(.+)/g, '<h6>$1</h6>')
    // Convert bold
    .replace(/\*\*(.+?)\*\*/g, '<strong>$1</strong>')
    // Convert italics
    .replace(/\*(.+?)\*/g, '<em>$1</em>')
    // Convert new lines to paragraphs
    .split('\n').map(line => line.trim() ? `<p>${line}</p>` : '').join('');
  
  // Store the HTML content to be applied after AI mode is exited
  // We need to use a short timeout to ensure the DOM has updated after exiting AI mode
  setTimeout(() => {
    if (!editorEl.value) return;
    
    // Set the content to the editor element
    editorEl.value.innerHTML = htmlContent;
    
    // Manually update the markdown output to reflect the AI-generated content
    updateMarkdown();
    
    console.log('AI content applied:', htmlContent);
    console.log('Markdown updated:', markdownOutput.value);
  }, 100);
};

// Reset editor to a blank state
const resetEditor = () => {
  // Check for unsaved changes
  if (hasUnsavedChanges.value) {
    // Show confirmation dialog
    newPromptDialog.value.show = true;
    return;
  }
  
  // If no unsaved changes, proceed directly
  performReset();
};

// Actual reset function that gets called after confirmation
const confirmNewPrompt = () => {
  performReset();
};

// Common reset implementation
const performReset = () => {
  if (!editorEl.value) return;
  
  // Reset to initial blank state
  editorEl.value.innerHTML = '<p>Comece a escrever seu prompt aqui...</p>';
  updateMarkdown();
  
  // Reset version tracking
  currentVersionIndex.value = null;
  lastSavedContent.value = markdownOutput.value;
  
  $q.notify({
    color: 'info',
    message: 'Editor reset to blank state',
    icon: 'refresh'
  });
};

// Initialize the editor
onMounted(() => {
  if (editorEl.value) {
    // Initialize with a sample prompt
    editorEl.value.innerHTML = '<p>Comece a escrever seu prompt aqui...</p>';
    updateMarkdown();
    
    // Set the initial content as the last saved content
    lastSavedContent.value = markdownOutput.value;
  }
});
</script>

<style scoped>
/* Card styling */
.editor-card, .markdown-card {
  width: 100%;
  height: auto;
  border-radius: 8px;
}

/* Toolbar styling */
.formatting-toolbar {
  border-top-left-radius: 8px;
  border-top-right-radius: 8px;
  border-bottom: 1px solid #e0e0e0;
}

/* Editor content area */
.editor-content {
  height: 500px;
  padding: 0;
}

.editor {
  height: 100%;
  padding: 16px;
  overflow-y: auto;
  outline: none;
  font-family: 'Roboto', sans-serif;
  font-size: 16px;
  line-height: 1.4;
}

.editor * {
  margin-top: 0;
  margin-bottom: 2px;
}

/* Title button styling */
.title-button {
  font-family: Georgia, serif;
  font-weight: bold;
  font-size: 16px;
  color: #6467F2;
}

/* Version control styling */
.version-selector {
  font-size: 14px;
}

.save-version-btn {
  font-size: 14px;
}

/* Markdown sections */
.markdown-header {
  padding: 12px 16px;
  background-color: #f8f9fa;
}

.markdown-content {
  padding: 16px;
}

.markdown-output {
  font-family: 'Courier New', monospace;
  font-size: 14px;
  background-color: #f5f5f5;
}

/* Styling for elements inside the editor */
.editor h6 {
  font-size: 18px;
  font-weight: bold;
  margin-top: 2px;
  margin-bottom: 2px;
  padding-top: 0;
  padding-bottom: 0;
  line-height: 1.2;
  color: #6467F2;
}

.editor p {
  margin-top: 0;
  margin-bottom: 4px;
  line-height: 1.4;
}

.editor ul {
  margin-left: 16px;
  padding-left: 16px;
}

.editor li {
  margin-bottom: 4px;
}

/* Color swatch for dropdown */
.color-swatch {
  width: 16px;
  height: 16px;
  border-radius: 4px;
  display: inline-block;
  border: 1px solid rgba(0, 0, 0, 0.1);
}

/* AI Editor styles */
.ai-editor-container {
  height: 100%;
  padding: 16px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
}

.ai-editor-container .text-subtitle1 {
  font-weight: 500;
  color: #6467F2;
}

.ai-editor-container .q-input {
  flex-grow: 1;
}

/* Add a placeholder animation for the loading state */
.ai-editor-container .q-spinner-dots {
  font-size: 2em;
}
</style> 