<template>
  <div class="row q-col-gutter-md">
    <!-- Editor Column (Left) -->
    <div class="col-6">
      <q-card class="editor-card q-mt-md shadow-5">
        <!-- Simple formatting toolbar -->
        <q-card-section class="formatting-toolbar q-py-sm bg-grey-2">
          <div class="row items-center">
            <q-btn flat dense size="md" icon="format_bold" @click="applyFormat('bold')" class="q-mr-xs" />
            <q-btn flat dense size="md" icon="format_italic" @click="applyFormat('italic')" class="q-mr-xs" />
            <q-btn flat dense size="md" icon="format_list_bulleted" @click="applyFormat('insertUnorderedList')" class="q-mr-xs" />
            
            <q-separator vertical inset class="q-mx-sm" />
            
            <!-- Title button -->
            <q-btn 
              flat 
              dense
              no-caps
              label="Tt"
              class="title-button q-mr-xs"
              @click="formatAsTitle"
              >
              <q-tooltip>Format line as title</q-tooltip>
            </q-btn>
            
            <!-- Normal text button -->
            <q-btn 
              flat 
              dense
              no-caps
              label="P"
              class="normal-button"
              @click="formatAsNormal"
              >
              <q-tooltip>Format as normal paragraph</q-tooltip>
            </q-btn>
          </div>
        </q-card-section>
        
        <!-- Editor area -->
        <q-card-section class="editor-content">
          <div 
            ref="editorEl" 
            class="editor" 
            contenteditable="true" 
            @input="updateMarkdown"
            @keydown="handleKeyDown"
            @keyup="updateMarkdown"
          ></div>
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
      
      <!-- Preview Card -->
      <q-card class="preview-card q-mt-md shadow-2">
        <q-card-section class="preview-header">
          <div class="text-subtitle1 text-weight-medium" style="color: #6467F2;">
            <q-icon name="visibility" class="q-mr-sm" />
            Formatted Preview
          </div>
          <div class="text-caption text-grey-7">
            How your prompt will look when rendered
          </div>
        </q-card-section>
        
        <q-separator />
        
        <q-card-section class="preview-content">
          <div class="preview-box markdown-body" v-html="renderedPreview"></div>
        </q-card-section>
      </q-card>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';
import turndown from 'turndown';

// Initialize the Turndown service for HTML to Markdown conversion
const turndownService = new turndown({
  headingStyle: 'atx',
  codeBlockStyle: 'fenced'
});

// Editor element reference
const editorEl = ref<HTMLElement | null>(null);

// Markdown output
const markdownOutput = ref('');

// Apply basic formatting
const applyFormat = (command: string) => {
  document.execCommand(command, false);
  updateMarkdown();
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
      
      // Restore original selection
      selection.removeAllRanges();
      selection.addRange(range);
    }
  }
  
  updateMarkdown();
};

// Format as normal paragraph
const formatAsNormal = () => {
  // Get the current selection
  const selection = window.getSelection();
  if (!selection || !selection.anchorNode) return;
  
  // Find the current element
  let currentNode: Node | null = selection.anchorNode;
  
  // If we're in a text node, get its parent
  if (currentNode.nodeType === Node.TEXT_NODE) {
    currentNode = currentNode.parentNode;
  }
  
  // Find the block-level element that contains this node
  while (currentNode && 
         currentNode.nodeName !== 'P' && 
         currentNode.nodeName !== 'H6' && 
         currentNode.nodeName !== 'DIV') {
    currentNode = currentNode.parentNode;
  }
  
  // Apply paragraph format only to this line
  if (currentNode) {
    // Save selection
    const range = selection.getRangeAt(0);
    
    // Select entire element
    const newRange = document.createRange();
    newRange.selectNodeContents(currentNode);
    selection.removeAllRanges();
    selection.addRange(newRange);
    
    // Apply paragraph format
    document.execCommand('formatBlock', false, 'p');
    
    // Restore original selection
    selection.removeAllRanges();
    selection.addRange(range);
  }
  
  updateMarkdown();
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

// Simple preview - converts markdown to basic HTML
const renderedPreview = computed(() => {
  let preview = markdownOutput.value;
  
  // Convert headers (######)
  preview = preview.replace(/^###### (.*$)/gm, '<h6>$1</h6>');
  
  // Convert bold
  preview = preview.replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>');
  
  // Convert italic
  preview = preview.replace(/\*(.*?)\*/g, '<em>$1</em>');
  
  // Convert lists (simple)
  preview = preview.replace(/^- (.*$)/gm, '<li>$1</li>');
  preview = preview.replace(/(<li>.*<\/li>)/gs, '<ul>$1</ul>');
  
  // Convert paragraphs
  preview = preview.replace(/^([^<].*?)$/gm, '<p>$1</p>');
  
  // Clean up empty paragraphs
  preview = preview.replace(/<p><\/p>/g, '');
  
  return preview;
});

// Initialize the editor
onMounted(() => {
  if (editorEl.value) {
    // Initialize with a sample prompt
    editorEl.value.innerHTML = '<p>Start typing your prompt here...</p>';
    updateMarkdown();
  }
});
</script>

<style scoped>
/* Card styling */
.editor-card, .markdown-card, .preview-card {
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
  line-height: 1.6;
}

/* Title button styling */
.title-button {
  font-family: Georgia, serif;
  font-weight: bold;
  font-size: 16px;
  color: #6467F2;
}

/* Normal text button styling */
.normal-button {
  font-family: Georgia, serif;
  font-size: 16px;
  color: #424242;
}

/* Markdown and preview sections */
.markdown-header, .preview-header {
  padding: 12px 16px;
  background-color: #f8f9fa;
}

.markdown-content, .preview-content {
  padding: 16px;
}

.markdown-output {
  font-family: 'Courier New', monospace;
  font-size: 14px;
  background-color: #f5f5f5;
}

.preview-box {
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 16px;
  min-height: 200px;
  background-color: white;
  overflow-y: auto;
}

/* Styling for elements inside the editor */
.editor h6 {
  font-size: 18px;
  font-weight: bold;
  margin-top: 8px;
  margin-bottom: 4px;
  color: #6467F2;
}

.editor p {
  margin-bottom: 8px;
}

.editor ul {
  margin-left: 16px;
  padding-left: 16px;
}

.editor li {
  margin-bottom: 4px;
}

/* Markdown preview styling */
.markdown-body h6 {
  font-size: 18px;
  font-weight: bold;
  margin-top: 8px;
  margin-bottom: 4px;
  color: #6467F2;
}
</style> 