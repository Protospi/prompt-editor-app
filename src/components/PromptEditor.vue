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
              
              <q-btn flat dense size="md" icon="format_list_bulleted" @click="applyFormat('insertUnorderedList')" class="title-button q-mr-xs" />
              
              <q-btn flat dense size="md" icon="format_bold" @click="applyFormat('bold')" class="title-button q-mr-xs" />
              
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

              <q-separator vertical inset class="q-mx-sm" />
              
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
              <!-- New Prompt button (moved to right side) -->
              <q-btn 
                flat 
                dense
                icon="add"
                size="md"
                class="title-button q-mr-xs"
                @click="createNewPrompt"
              >
                <q-tooltip>Create new prompt document</q-tooltip>
              </q-btn>
              
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
                :color="hasUnsavedChanges ? 'positive' : 'grey-6'" 
                icon="save" 
                @click="saveVersion"
                class="save-version-btn"
                :disable="!hasUnsavedChanges"
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
            @mouseup="handleMouseSelection"
            @click="handleMouseSelection"
            @focus="ensureProperFormatting"
            @paste="handlePaste"
            @blur="ensureProperFormatting"
            @copy="handleCopy"
          ></div>
          
          <!-- AI editor mode when AI mode is on -->
          <div v-if="aiModeActive" class="ai-editor-container">
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
          <q-btn flat label="Create New" color="negative" @click="confirmNewPrompt" v-close-popup />
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
import { ref, onMounted, computed, onBeforeUnmount } from 'vue';
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

// Store the last copied markdown segment
const lastCopiedMarkdown = ref('');

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
  
  if (currentVersionIndex.value !== null) {
    return 'Save changes to current version';
  }
  
  if (allSlotsFilled.value) {
    return 'All slots filled. Delete a version first.';
  }
  
  return 'Save as a new version';
};

// Apply basic formatting
const applyFormat = (command: string) => {
  // Handle special cases for lists when applying to multiple paragraphs
  const selection = window.getSelection();
  if (selection && selection.rangeCount > 0 && !selection.isCollapsed) {
    // For list commands, we need special handling for multi-paragraph selections
    if (command === 'insertUnorderedList' || command === 'insertOrderedList') {
      document.execCommand(command, false);
      updateMarkdown();
      return;
    }
    
    // For block formatting commands, we can just execute directly
    document.execCommand(command, false);
    updateMarkdown();
    return;
  }
  
  // Default case for cursor position or single element selection
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
    // Apply color to current selection
    document.execCommand('foreColor', false, colorValue);
    updateMarkdown();
  }
};

// Format as title (H6)
const formatAsTitle = () => {
  // Get the current selection
  const selection = window.getSelection();
  if (!selection) return;
  
  // Check if we have a non-collapsed selection (multiple elements or text spans)
  if (selection.rangeCount > 0 && !selection.isCollapsed) {
    // Apply heading directly to the selection
    document.execCommand('formatBlock', false, 'h6');
    
    // Update markdown to reflect changes
    updateMarkdown();
    return;
  }
  
  // Original code for cursor position or collapsed selection
  // Special case for first line or empty selection - select the whole paragraph
  if (selection.isCollapsed || !selection.rangeCount) {
    // Find the containing block element
    let container = selection.anchorNode;
    if (!container) return;
    
    // If in text node, get parent
    if (container.nodeType === Node.TEXT_NODE) {
      container = container.parentNode;
    }
    
    // If this is a valid block container, select its content
    if (container && (container.nodeName === 'P' || container.nodeName === 'H6')) {
      const range = document.createRange();
      range.selectNodeContents(container);
      selection.removeAllRanges();
      selection.addRange(range);
    }
  }
  
  // Check if selection is now valid
  if (!selection.anchorNode) return;
  
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
    // We'll use a more direct and reliable approach:
    
    // 1. First identify the paragraph containing the selection
    let paragraphNode = currentNode;
    while (paragraphNode && paragraphNode.nodeName !== 'P' && paragraphNode.nodeName !== 'DIV') {
      paragraphNode = paragraphNode.parentNode;
    }
    
    // 2. If we found a valid container, apply the heading
    if (paragraphNode) {
      // Select the entire paragraph to ensure consistent formatting
      const range = document.createRange();
      range.selectNodeContents(paragraphNode);
      selection.removeAllRanges();
      selection.addRange(range);
      
      // Apply heading format (H6)
      document.execCommand('formatBlock', false, 'h6');
      
      // Get the newly created h6 element
      // This requires finding it again after the format change
      const newHeading = selection.anchorNode;
      if (newHeading) {
        // If this is an H6 element, apply the custom styles
        if (newHeading.nodeName === 'H6') {
          const headingEl = newHeading as HTMLElement;
          
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
      }
    }
  }
  
  // Update markdown immediately to reflect changes
  updateMarkdown();
};

// Handle keydown events for tabs in lists and enter in headings
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
  
  // Handle Enter key to prevent heading style from being applied to the next line
  if (event.key === 'Enter') {
    // Always prevent the default Enter behavior to have full control
    event.preventDefault();
    
    const selection = window.getSelection();
    if (!selection || !selection.anchorNode) return;
    
    // Find the current node and its parent element
    let currentNode: Node | null = selection.anchorNode;
    
    // If we're in a text node, get its parent
    if (currentNode.nodeType === Node.TEXT_NODE) {
      currentNode = currentNode.parentNode;
    }
    
    // Check if we're in a heading element or any of its parents
    let isInHeading = false;
    let headingNode: Node | null = null;
    let tempNode = currentNode;
    
    while (tempNode) {
      if (tempNode.nodeName === 'H6') {
        isInHeading = true;
        headingNode = tempNode;
        break;
      }
      tempNode = tempNode.parentNode;
    }
    
    // Explicitly create a new paragraph after either a heading or regular paragraph
    if (editorEl.value) {
      // First, insert a line break to create a new paragraph
      document.execCommand('insertParagraph', false);
      
      // Immediately force paragraph format on the newly created paragraph
      document.execCommand('formatBlock', false, 'p');
      
      // Get the current node again after inserting the paragraph
      currentNode = selection.anchorNode;
      if (currentNode && currentNode.nodeType === Node.TEXT_NODE) {
        currentNode = currentNode.parentNode;
      }
      
      // If we found a node and it's not already a paragraph, force paragraph format again
      if (currentNode && currentNode.nodeName !== 'P') {
        document.execCommand('formatBlock', false, 'p');
      }
      
      // Update the markdown output
      updateMarkdown();
    }
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
    // Special handling for editor state
    const firstChild = editorEl.value.firstChild;
    
    // If the editor has content but isn't properly wrapped in blocks, fix it
    if (firstChild && firstChild.nodeType === Node.TEXT_NODE) {
      // Wrap loose text in paragraphs for proper formatting
      const text = editorEl.value.textContent || '';
      editorEl.value.innerHTML = `<p>${text}</p>`;
    }
    
    // Continue with normal formatting checks
    ensureProperFormatting();
    
    // Update the markdown output
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

// Save current version (either update existing or create new if none exists)
const saveVersion = () => {
  // Don't allow saving if there are no changes
  if (!hasUnsavedChanges.value) {
    return;
  }
  
  // If there's a current version, update it
  if (currentVersionIndex.value !== null) {
    updateCurrentVersion();
    return;
  }
  
  // Otherwise, this is a new prompt that hasn't been saved yet
  // Check if all slots are filled before trying to save a new version
  if (allSlotsFilled.value) {
    $q.notify({
      color: 'warning',
      message: 'All slots are filled. Please delete a version first or update an existing one.',
      icon: 'warning',
      timeout: 3000
    });
    return;
  }
  
  // Find the first empty slot
  const emptySlotIndex = promptVersions.value.findIndex(v => v === null);
  
  if (emptySlotIndex !== -1) {
    // We have an empty slot, save to it
    createNewVersion(emptySlotIndex);
  }
};

// Update the current version with the latest content
const updateCurrentVersion = () => {
  if (!editorEl.value || currentVersionIndex.value === null) return;
  
  const version = promptVersions.value[currentVersionIndex.value];
  if (!version) return;
  
  // Update the version content
  version.content = markdownOutput.value;
  version.html = editorEl.value.innerHTML;
  // We don't update the name or timestamp to preserve the version identity
  
  // Update saved content reference
  lastSavedContent.value = markdownOutput.value;
  
  $q.notify({
    color: 'positive',
    message: `Updated ${version.name}`,
    icon: 'save'
  });
};

// Create a new version in the specified slot
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

// Load a version from the version list
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
const submitAiPrompt = async () => {
  if (!aiPromptInput.value.trim()) {
    $q.notify({
      color: 'negative',
      message: 'Please enter instructions for the AI',
      icon: 'warning'
    });
    return;
  }
  
  isProcessingAiRequest.value = true;
  
  try {
    // Prepare the payload
    const payload = {
      payload: {
        prompt: `This is my current prompt:\n${markdownOutput.value}\n\nThis is my instructions:\n${aiPromptInput.value}`
      }
    };
    
    // Get API config from env vars
    const apiUrl = import.meta.env.VITE_API_URL;
    const apiToken = import.meta.env.VITE_API_TOKEN;
    
    // Make the API request
    const response = await fetch(apiUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${apiToken}`
      },
      body: JSON.stringify(payload)
    });
    
    if (!response.ok) {
      throw new Error(`API error: ${response.status}`);
    }
    
    const data = await response.json();
    
    // First exit AI mode
    aiModeActive.value = false;
    
    // Then apply the response to the editor
    if (data && data.prompt) {
      applyAiGeneratedPrompt(data.prompt);
      
      $q.notify({
        color: 'positive',
        message: 'AI suggestion applied to editor. Review and save if you like it.',
        icon: 'auto_awesome',
        timeout: 3000,
        position: 'top'
      });
    } else {
      throw new Error('Invalid API response format');
    }
  } catch (error) {
    console.error('Error calling AI API:', error);
    
    $q.notify({
      color: 'negative',
      message: `Error generating AI suggestion: ${error instanceof Error ? error.message : 'Unknown error'}`,
      icon: 'error',
      timeout: 5000,
      position: 'top'
    });
    
    // Stay in AI mode on error
    aiModeActive.value = true;
  } finally {
    // Reset the AI processing state
    isProcessingAiRequest.value = false;
  }
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

// Create a new prompt (replaces resetEditor)
const createNewPrompt = () => {
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
  
  // Reset version tracking - this makes it a new unsaved prompt
  currentVersionIndex.value = null;
  lastSavedContent.value = markdownOutput.value;
  
  $q.notify({
    color: 'info',
    message: 'Created new blank prompt',
    icon: 'add'
  });
};

// Handle mouse selection to ensure formatting is appropriate
const handleMouseSelection = () => {
  const selection = window.getSelection();
  if (!selection) return;
  
  // If selection is empty or just a cursor position, ensure proper paragraph formatting
  if (!selection.rangeCount || selection.isCollapsed) {
    ensureProperFormatting();
    updateMarkdown();
    return;
  }
  
  // Selection exists - just update markdown without modifying the selection
  updateMarkdown();
};

// Ensure that the current cursor position/selection has proper formatting
const ensureProperFormatting = () => {
  // If no selection exists yet, exit
  const selection = window.getSelection();
  if (!selection) return;
  
  // Special handling for empty editor or initial paragraph
  if (editorEl.value) {
    // Check if we're on the initial element or an empty editor
    const firstChild = editorEl.value.firstChild;
    
    // If editor is empty or only has a single paragraph, ensure it's properly formatted
    if (!firstChild) {
      // Create an initial paragraph if editor is completely empty
      const p = document.createElement('p');
      p.innerHTML = '<br>'; // Empty paragraph needs a br to be visible
      editorEl.value.appendChild(p);
      
      // Put cursor in it
      const range = document.createRange();
      range.setStart(p, 0);
      range.collapse(true);
      selection.removeAllRanges();
      selection.addRange(range);
      return;
    }
    
    // Ensure there's always a first paragraph (not heading)
    if (firstChild && firstChild.nodeName !== 'P' && firstChild.nodeName !== 'H6') {
      // Wrap content in paragraph if needed
      document.execCommand('formatBlock', false, 'p');
    }
  }
  
  // If there's no anchor node, exit
  if (!selection.anchorNode) return;
  
  // Get current node
  let currentNode: Node | null = selection.anchorNode;
  
  // If we're in a text node, get its parent element
  if (currentNode.nodeType === Node.TEXT_NODE) {
    currentNode = currentNode.parentNode;
  }
  
  // Only apply default paragraph formatting if the cursor is positioned and not in a heading
  // and we're not working with a selection
  if (selection.isCollapsed && currentNode && currentNode.nodeName !== 'H6') {
    // Force paragraph format for non-heading nodes only when cursor is positioned (not selecting text)
    document.execCommand('formatBlock', false, 'p');
  }
};

// Add custom copy handler to the editor
const handleCopy = (event: ClipboardEvent) => {
  // Prevent the default copy behavior
  event.preventDefault();
  
  // Get the current selection
  const selection = window.getSelection();
  if (!selection || selection.rangeCount === 0) return;
  
  // Get the selected content
  const range = selection.getRangeAt(0);
  if (!range) return;
  
  // Create a temporary div to hold the selected HTML
  const tempDiv = document.createElement('div');
  tempDiv.appendChild(range.cloneContents());
  
  // Process colored text before converting to markdown
  // Find all spans with color style and add our custom color tags
  const coloredSpans = tempDiv.querySelectorAll('span[style*="color"]');
  coloredSpans.forEach(span => {
    // Extract color value
    const style = span.getAttribute('style') || '';
    const colorMatch = style.match(/color:\s*([^;]+)/i);
    if (colorMatch && colorMatch[1]) {
      const colorValue = colorMatch[1].trim();
      
      // Create wrapper with our custom color syntax
      const wrapper = document.createElement('span');
      wrapper.setAttribute('data-color', colorValue);
      
      // Move the span's children to the wrapper
      while (span.firstChild) {
        wrapper.appendChild(span.firstChild);
      }
      
      // Replace the original span with our wrapper
      span.parentNode?.replaceChild(wrapper, span);
    }
  });
  
  // Convert the HTML to markdown using turndown
  let selectedMarkdown = turndownService.turndown(tempDiv.innerHTML);
  
  // Process our custom color tags
  selectedMarkdown = selectedMarkdown.replace(
    /\[([^\]]+)\]\{data-color="([^"]+)"\}/g, 
    '{color:$2}$1{/color}'
  );
  
  // Save the markdown for internal use
  lastCopiedMarkdown.value = selectedMarkdown;
  
  // Add both plain text and HTML versions to clipboard
  if (event.clipboardData) {
    event.clipboardData.setData('text/plain', selectedMarkdown);
    event.clipboardData.setData('text/html', tempDiv.innerHTML);
  }
  
  console.log('Copied markdown with colors:', selectedMarkdown);
};

// Handle paste operations to support markdown content
const handlePaste = (event: ClipboardEvent) => {
  // Prevent the default paste behavior
  event.preventDefault();
  
  // First check if we have content in clipboard
  if (!event.clipboardData) return;
  
  // Try to get HTML content first for better formatting preservation
  const pastedHtml = event.clipboardData.getData('text/html');
  if (pastedHtml) {
    // Clean and insert HTML directly
    document.execCommand('insertHTML', false, sanitizeHtml(pastedHtml));
    updateMarkdown();
    return;
  }
  
  // Get the markdown or plain text content
  const pastedText = event.clipboardData.getData('text/plain');
  if (!pastedText) return;
  
  // Check if this looks like markdown (contains markdown syntax)
  const hasMarkdownSyntax = /#{1,6}\s|[*_]|^\s*[-+*]\s|-{3,}|`{1,3}|>\s/.test(pastedText);
  
  if (hasMarkdownSyntax) {
    // Try to convert markdown to HTML
    try {
      // Create a temporary element to convert markdown to HTML
      let html = pastedText
        // Convert headings (h6 for our app)
        .replace(/^#{1,6}\s+(.+)$/gm, '<h6>$1</h6>')
        // Convert bold
        .replace(/\*\*(.+?)\*\*/g, '<strong>$1</strong>')
        .replace(/__(.+?)__/g, '<strong>$1</strong>') // underscore bold
        // Convert italic
        .replace(/\*(.+?)\*/g, '<em>$1</em>')
        .replace(/_(.+?)_/g, '<em>$1</em>') // underscore italic
        // Convert bullet lists - handle multiple bullet types
        .replace(/^\s*[-+*]\s+(.+)$/gm, '<li>$1</li>');
      
      // Process lists - wrap consecutive li elements in ul
      let processedHtml = '';
      let inList = false;
      html.split('\n').forEach(line => {
        if (line.includes('<li>')) {
          if (!inList) {
            processedHtml += '<ul>';
            inList = true;
          }
          processedHtml += line;
        } else {
          if (inList) {
            processedHtml += '</ul>';
            inList = false;
          }
          // Only wrap non-formatted lines in paragraphs
          if (line.trim() && !line.includes('<h6>')) {
            processedHtml += `<p>${line}</p>`;
          } else {
            processedHtml += line;
          }
        }
      });
      
      // Close any open lists
      if (inList) {
        processedHtml += '</ul>';
      }
      
      // Check for color syntax - simplified custom format: {color:red}text{/color}
      processedHtml = processedHtml.replace(/{color:([\w#]+)}(.+?){\/color}/g, 
        '<span style="color:$1">$2</span>');
      
      // Insert the HTML at the cursor position
      document.execCommand('insertHTML', false, processedHtml);
      
      // Update markdown output
      updateMarkdown();
      return;
    } catch (error) {
      console.error('Error processing markdown:', error);
      // Fall through to plain text handling
    }
  }
  
  // For plain text or if markdown processing failed
  document.execCommand('insertText', false, pastedText);
  
  // Update markdown and check formatting
  updateMarkdown();
  ensureProperFormatting();
};

// Helper to sanitize HTML input (basic implementation)
const sanitizeHtml = (html: string): string => {
  // Create a temporary div
  const tempDiv = document.createElement('div');
  tempDiv.innerHTML = html;
  
  // Remove any script tags or event handlers
  const scripts = tempDiv.querySelectorAll('script');
  scripts.forEach(script => script.remove());
  
  // Remove potentially dangerous attributes
  const allElements = tempDiv.querySelectorAll('*');
  allElements.forEach(el => {
    // Remove event handlers
    for (let i = el.attributes.length - 1; i >= 0; i--) {
      const attrName = el.attributes[i]?.name;
      if (attrName && (
        attrName.startsWith('on') || 
        (attrName === 'href' && el.attributes[i]?.value?.startsWith('javascript:'))
      )) {
        el.removeAttribute(attrName);
      }
    }
  });
  
  return tempDiv.innerHTML;
};

// Initialize the editor
onMounted(() => {
  if (editorEl.value) {
    // Initialize with a sample prompt
    editorEl.value.innerHTML = '<p>Comece a escrever seu prompt aqui...</p>';
    
    // Add event listener for copy
    editorEl.value.addEventListener('copy', handleCopy);
    
    // Ensure proper formatting is applied
    setTimeout(() => {
      // Focus the editor
      editorEl.value?.focus();
      
      // Ensure proper formatting
      ensureProperFormatting();
      
      // Update markdown
      updateMarkdown();
      
      // Set the initial content as the last saved content
      lastSavedContent.value = markdownOutput.value;
      
      // Set current version to null (new unsaved document)
      currentVersionIndex.value = null;
      
      // Create a cursor position at the beginning of the text
      const selection = window.getSelection();
      if (selection && editorEl.value?.firstChild) {
        const range = document.createRange();
        range.setStart(editorEl.value.firstChild, 0);
        range.collapse(true);
        selection.removeAllRanges();
        selection.addRange(range);
      }
    }, 100);
  }
});

// Cleanup event listeners
onBeforeUnmount(() => {
  if (editorEl.value) {
    // Remove copy event listener
    editorEl.value.removeEventListener('copy', handleCopy);
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