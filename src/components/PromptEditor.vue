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

              <!-- New Diagram Button -->
              <q-btn 
                flat 
                dense
                :icon="diagramModeActive ? 'edit' : 'account_tree'"
                size="md"
                :color="diagramModeActive ? 'positive' : '#6467F2'"
                class="title-button q-mr-xs"  
                @click="submitDiagramPrompt"
              >
                <q-tooltip>{{ diagramModeActive ? 'Return to editor' : 'Create flow diagram' }}</q-tooltip>
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
              
              <!-- Language Selector -->
              <q-select
                v-model="selectedLanguage"
                :options="currentVersionIndex === null ? languageOptions : availableLanguageOptions"
                dense
                outlined
                emit-value
                map-options
                style="min-width: 66px; max-width: 66px;"
                class="q-mr-sm language-selector"
                @update:model-value="handleLanguageChange"
                color="primary"
                bg-color="white"
              >
                <template v-slot:selected>
                  <div class="row items-center justify-center">
                    <span class="flag-icon">{{ getFlagForLanguage(selectedLanguage) }}</span>
                  </div>
                </template>
                <template v-slot:no-option>
                  <q-item>
                    <q-item-section class="text-grey">
                      All languages already saved for this version
                    </q-item-section>
                  </q-item>
                </template>
                <template v-slot:option="scope">
                  <q-item
                    v-bind="scope.itemProps"
                    :active="scope.opt.value === selectedLanguage"
                    active-class="bg-primary-1"
                  >
                    <q-item-section>
                      <div class="row items-center">
                        <span class="flag-icon">{{ getFlagForLanguage(scope.opt.value) }}</span>
                        <span class="q-ml-xs">{{ scope.opt.label.replace(getFlagForLanguage(scope.opt.value), '').trim() }}</span>
                      </div>
                    </q-item-section>
                  </q-item>
                </template>
                <q-tooltip>
                  <template v-if="currentVersionIndex !== null">
                    <div v-if="isLanguageAlreadySaved">
                      {{ selectedLanguage === currentLoadedLanguage ? 'Currently editing this language' : 'This language already exists for this version. Select to edit it.' }}
                    </div>
                    <div v-else>
                      Select this language to create a new variant for this version
                    </div>
                  </template>
                  <template v-else>
                    Select a language for your prompt
                  </template>
                </q-tooltip>
              </q-select>
              
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
                      <q-item-section side>
                        <div class="row items-center">
                          <q-btn flat round dense icon="edit" size="xs" @click.stop="promptForRename(index)">
                            <q-tooltip>Rename</q-tooltip>
                          </q-btn>
                          <q-btn flat round dense icon="delete" size="xs" color="negative" @click.stop="promptForDelete(index)">
                            <q-tooltip>Delete version</q-tooltip>
                          </q-btn>
                          <q-btn flat round dense icon="keyboard_arrow_right" size="xs">
                            <q-tooltip>Language options</q-tooltip>
                            <q-menu anchor="top right" self="top left" class="language-menu" square :offset="[10, 0]">
                              <q-list class="submenu-list">
                                
                                <!-- Languages list - only show languages that actually have content -->
                                <template v-for="langCode in getExistingLanguagesForVersion(index)" :key="langCode">
                                  <q-item 
                                    clickable 
                                    v-close-popup
                                    @click="loadLanguageVariant(index, langCode)"
                                    :active="currentVersionIndex === index && currentLoadedLanguage === langCode"
                                    active-class="bg-primary-1"
                                    dense
                                  >
                                    <q-item-section>
                                      <div class="row items-center">
                                        <span class="flag-icon q-mr-xs">{{ getFlagForLanguage(langCode) }}</span>
                                        <span>{{ getLanguageLabel(langCode).replace(getFlagForLanguage(langCode), '').trim() }}</span>
                                        <span v-if="langCode === 'pt-BR'" class="text-caption text-grey-8 q-ml-xs">(Default)</span>
                                      </div>
                                    </q-item-section>
                                    <q-item-section side class="row items-center no-wrap">
                                      <q-icon name="check" size="xs" color="primary" v-if="currentVersionIndex === index && currentLoadedLanguage === langCode" />
                                      <q-btn 
                                        v-if="langCode !== 'pt-BR'"
                                        flat 
                                        round 
                                        dense 
                                        icon="delete" 
                                        size="xs" 
                                        color="negative"
                                        @click.stop="deleteLanguageVariant(index, langCode)"
                                      >
                                        <q-tooltip>Delete language variant</q-tooltip>
                                      </q-btn>
                                    </q-item-section>
                                  </q-item>
                                </template>

                                <!-- Show a message if no languages -->
                                <q-item v-if="getExistingLanguagesForVersion(index).length === 0">
                                  <q-item-section class="text-caption text-grey">
                                    No languages available
                                  </q-item-section>
                                </q-item>
                              </q-list>
                            </q-menu>
                          </q-btn>
                        </div>
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
                <q-tooltip>
                  <template v-if="!hasUnsavedChanges">
                    <span v-if="isLanguageAlreadySaved && selectedLanguage !== currentLoadedLanguage">
                      This language variant already exists. Make changes to enable saving.
                    </span>
                    <span v-else>
                      Make changes or select a new language to enable saving
                    </span>
                  </template>
                  <template v-else>
                    {{ getSaveButtonTooltip() }}
                  </template>
                </q-tooltip>
              </q-btn>
            </div>
          </div>
        </q-card-section>
        
        <!-- Editor area -->
        <q-card-section class="editor-content">
          <!-- Regular editor when AI mode is off -->
          <div 
            v-if="!aiModeActive && !diagramModeActive"
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
            @compositionstart="handleCompositionStart"
            @compositionend="handleCompositionEnd"
          ></div>
          
          <!-- AI editor mode when AI mode is on -->
          <div v-if="aiModeActive" class="ai-editor-container">
            <div class="text-subtitle1 q-mb-md">
              <q-icon name="smart_toy" class="q-mr-xs" /> AI Prompt Assistant
            </div>
            
            <p class="text-body2 q-mb-sm">Describe the prompt you'd like to create or edit like in the examples below:</p>
            <p class="text-caption text-grey-8 q-mb-md">
              <ul>
              <li>Write me an extraction prompt to extract variables from a conversation.</li>
              <li>Write me an conversational agent prompt to talk about a hotel services.</li>
            </ul>
            </p>
            
            <q-input
              v-model="aiPromptInput"
              type="textarea"
              outlined
              rows="10"
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

          <!-- Diagram editor mode when diagram mode is on -->
          <div v-if="diagramModeActive" class="diagram-container">
            <div class="text-subtitle1 q-mb-md row items-center justify-between">
              <div class="row items-center">
                <q-icon name="account_tree" class="q-mr-xs" /> Flow Diagram
              </div>
              <!-- Zoom controls moved to title row -->
              <div class="diagram-zoom-controls">
                <span class="zoom-level">{{ Math.round(diagramZoom * 100) }}%</span>
                <q-btn
                  flat
                  dense
                  round
                  icon="add"
                  color="grey-7"
                  @click="zoomIn"
                  size="sm"
                >
                  <q-tooltip>Zoom in</q-tooltip>
                </q-btn>
                <q-btn
                  flat
                  dense
                  round
                  icon="remove"
                  color="grey-7"
                  @click="zoomOut"
                  size="sm"
                >
                  <q-tooltip>Zoom out</q-tooltip>
                </q-btn>
              </div>
            </div>
            
            <div class="diagram-content q-mb-md" ref="diagramEl">
              <!-- Zoom controls moved to title row -->
            </div>
            
            <!-- Action buttons at the bottom -->
            <div class="row justify-end q-mt-md">
              <q-btn 
                label="Back to Editor" 
                color="grey-7" 
                @click="toggleDiagramMode" 
                flat
                class="q-mr-sm"
                icon="arrow_back"
              >
                <q-tooltip>Return to the regular editor without changes</q-tooltip>
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
          <div class="row items-center justify-between">
            <!-- Left tab -->
            <div class="col-6">
              <q-btn 
                flat 
                no-caps
                class="full-width tab-button"
                :color="activeTab === 'markdown' ? 'primary' : 'grey-7'"
                :class="{'active-tab': activeTab === 'markdown'}"
                @click="activeTab = 'markdown'"
              >
                <div class="row items-center">
                  <q-icon name="code" class="q-mr-sm" />
                  <div class="text-subtitle1 text-weight-medium">Markdown Output</div>
                </div>
              </q-btn>
            </div>
            
            <!-- Right tab -->
            <div class="col-6">
              <q-btn 
                flat 
                no-caps
                class="full-width tab-button"
                :color="activeTab === 'versioning' ? 'primary' : 'grey-7'"
                :class="{'active-tab': activeTab === 'versioning'}"
                @click="activeTab = 'versioning'"
              >
                <div class="row items-center">
                  <q-icon name="history" class="q-mr-sm" />
                  <div class="text-subtitle1 text-weight-medium">Versioning Prompt</div>
                </div>
              </q-btn>
            </div>
          </div>
        </q-card-section>
        
        <q-separator />
        
        <q-card-section class="markdown-content">
          <!-- Markdown output view -->
          <div v-if="activeTab === 'markdown'">
            <div class="text-caption text-grey-7 q-mb-sm">
              This is the markdown that will be sent to the AI model
            </div>
            <q-input
              type="textarea"
              v-model="markdownOutput"
              readonly
              filled
              autogrow
              class="markdown-output"
              bg-color="#f5f5f5"
            />
          </div>
          
          <!-- Versioning data view -->
          <div v-if="activeTab === 'versioning'">
            <div class="row items-center justify-between q-mb-sm">
              <div class="text-caption">
                <div class="text-primary text-weight-medium">Prompt Version Storage Structure</div>
                <div class="text-grey-7">
                  This view shows how prompt versions are stored, including the content for each language variant
                </div>
              </div>
              <div class="row items-center">
                <q-btn 
                  flat 
                  dense 
                  icon="file_download" 
                  color="primary" 
                  size="sm"
                  class="q-mr-sm"
                  @click="downloadVersionData"
                >
                  <q-tooltip>Export as JSON</q-tooltip>
                </q-btn>
                <q-toggle
                  v-model="showFullPromptContent"
                  label="Show full content"
                  color="primary"
                  size="xs"
                  dense
                  class="q-mr-sm"
                />
                <q-btn 
                  flat 
                  round 
                  dense 
                  icon="refresh" 
                  color="primary" 
                  size="sm"
                  @click="refreshVersionData"
                >
                  <q-tooltip>Refresh version data</q-tooltip>
                </q-btn>
              </div>
            </div>
            <pre class="version-data-display q-pa-md" v-html="highlightedVersionData"></pre>
          </div>
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
import mermaid from 'mermaid';

// Initialize Quasar
const $q = useQuasar();

// Helper to get the API token - tries environment variables first, then falls back to a hardcoded value
// from the cURL command for development purposes
const getApiToken = () => {
  // First try the environment variable
  const envToken = import.meta.env.VITE_API_TOKEN;
  if (envToken) {
    console.log('Using token from environment variable');
    return envToken;
  }
  
  // If that fails, use the token from the cURL command as fallback
  console.log('Using hardcoded fallback token');
  return 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ2ZXJzaW9uIjoxNDQ2NjU3LCJncm91cElkIjoiNjc0NGRkZjk5MGM2MjIxZjUyNzY4MjkwIiwiYWNjb3VudElkIjoiNjc0NGRlMTk5MGM2MjIxZjUyNzY4OWIyIiwidW5pdHMiOltdLCJhY2NvdW50cyI6W10sIl9pZCI6IjYyOWEzZDI0MmVmMTUzMGIxODA4Y2Y5YSIsIm5hbWUiOiJQZWRybyBMb2VzIiwiZW1haWwiOiJwZWRyby5sb2VzQHNtYXJ0dGFsa3MuYWkiLCJhdmF0YXIiOiJodHRwczovL2NvbnRlbnQuc21hcnR0YWxrcy5haS82MjRmNDU2OTJjZDExZTM0NTE4MGJmZDEvMTczMzkzOTU5MDE2Ny1ibG9iIiwicm9sZSI6Ikd1aWRlciIsImRlcGFydG1lbnQiOiI2MmYzZTA4NGRkZDBjOGUzODIyMjg5YjkiLCJwZXJtaXNzaW9uc0dyb3VwcyI6WyI2MzJiNWQ1ODUxN2I4N2EzNTQ5MGIzYmEiXSwic2V0dGluZ3MiOnsiaXNNb2JpbGUiOmZhbHNlLCJ2b2x1bWUiOjAuNSwiaXNCZXRhVGVzdGVyIjpmYWxzZSwiaGFzRmxvd0FjY2VzcyI6ZmFsc2UsImhhc0FnZW50QWNjZXNzIjpmYWxzZSwidGltZXpvbmUiOiJFdGMvR01UKzMiLCJsb2NhbGl6YXRpb24iOiJwdC1CUiIsInNob3dOb3RpZmljYXRpb24iOnRydWUsImFsbEludGVyYWN0aW9ucyI6dHJ1ZSwiZ3JvdXBJbnRlcmFjdGlvbnMiOnRydWUsIm15UXVldWVzIjp0cnVlLCJvcmRlckJ5IjoiZGVjcmVhc2luZ0Fycml2YWxEYXRlIiwic2hvd0NoYW5uZWwiOmZhbHNlLCJzaG93RGl2aXNpb25CeVF1ZXVlcyI6ZmFsc2UsInNob3dOb3RpZmljYXRpb25Db3VudCI6ZmFsc2UsInNob3dUYWdzIjpmYWxzZSwic2hvd1RpbWVyIjpmYWxzZSwic2hvd1RvdGFsSW50ZXJhY3Rpb25zIjpmYWxzZSwid2FybmluZ0xpbWl0IjoxfSwibG9jYWxpemF0aW9uIjoicHQtQlIiLCJ0aW1lem9uZSI6IkV0Yy9HTVQrMyIsImlzR3VpZGVyIjp0cnVlLCJtb2R1bGVzUGVybWlzc2lvbnMiOlt7Im5hbWUiOiJhZG1pbiIsInBlcm1pc3Npb24iOiJ3cml0ZSIsIl9pZCI6IjYzMmI1ZDU4NTE3Yjg3YTM1NDkwYjNiYyJ9XSwiY2FuU2VlQWxsUXVldWVzIjp0cnVlLCJjYW5Bc3NpZ24iOnRydWUsImNhblRyYW5zZmVyIjp0cnVlLCJjYW5GaW5pc2giOnRydWUsImNhbEhzbSI6dHJ1ZSwiY2FuUmVtb3ZlQ29udGFjdCI6dHJ1ZSwiY2FuRXhwb3J0SW50ZXJhY3Rpb25zIjp0cnVlLCJjYW5TZWVBbGxJbnRlcmFjdGlvbnMiOnRydWUsImNhblNlbmRBdWRpbyI6dHJ1ZSwiZW50aXRpZXMiOltdLCJyZWNlaXZlQXV0b21hdGVkQXR0ZW5kYW5jZXMiOmZhbHNlLCJhdmFpbGFiaWxpdHkiOiJvZmZsaW5lIiwicHJvZmlsZSI6ImF0dGVuZGFudCIsImVtYWlsU2lnbmF0dXJlIjoiLS0tPGJyPjxicj5QZWRybyBMb2VzIiwiaWF0IjoxNzQ0MjI5Mjc2LCJleHAiOjE3NDQ4MzQwNzZ9.O-O3y0pG28fFBieEL6ClfOnrYrYPMVwVfBXJIdwEy00';
};

// Initialize Mermaid
onMounted(() => {
  mermaid.initialize({
    startOnLoad: false,
    theme: 'default',
    securityLevel: 'loose',
  });
});

// Tab switching for markdown/versioning view
const activeTab = ref('markdown');

// Toggle for showing full prompt content in version data
const showFullPromptContent = ref(true);

// Diagram mode state
const diagramModeActive = ref(false);
const diagramEl = ref<HTMLElement | null>(null);
const isProcessingDiagramRequest = ref(false);
// Diagram zoom state
const diagramZoom = ref(1);
const minZoom = 0.5;
const maxZoom = 2.5;
const zoomStep = 0.1;
// Diagram drag state
const isDragging = ref(false);
const dragStartPos = ref({ x: 0, y: 0 });
const currentTranslate = ref({ x: 0, y: 0 });

// Zoom functions
const zoomIn = () => {
  if (diagramZoom.value < maxZoom) {
    diagramZoom.value = Math.min(diagramZoom.value + zoomStep, maxZoom);
    applyZoom();
    console.log('Zoom in:', diagramZoom.value);
  }
};

const zoomOut = () => {
  if (diagramZoom.value > minZoom) {
    diagramZoom.value = Math.max(diagramZoom.value - zoomStep, minZoom);
    applyZoom();
    console.log('Zoom out:', diagramZoom.value);
  }
};

const applyZoom = () => {
  if (diagramEl.value) {
    const svgElement = diagramEl.value.querySelector('.mermaid svg');
    
    if (svgElement) {
      // Apply zoom and translation to the SVG
      (svgElement as HTMLElement).style.transform = 
        `translate(${currentTranslate.value.x}px, ${currentTranslate.value.y}px) scale(${diagramZoom.value})`;
      (svgElement as HTMLElement).style.transformOrigin = 'center center';
      
      // Update zoom display if needed
      console.log(`Current zoom: ${Math.round(diagramZoom.value * 100)}%`);
    }
  }
};

// Drag functions
const startDrag = (event: MouseEvent) => {
  if (!diagramModeActive.value) return;
  
  // Only initiate drag if we're clicking on the diagram content but not on any diagram node
  const target = event.target as HTMLElement;
  if (target.closest('.diagram-zoom-controls')) return;
  
  isDragging.value = true;
  dragStartPos.value = { x: event.clientX, y: event.clientY };
  
  // Set cursor to indicate dragging
  if (diagramEl.value) {
    diagramEl.value.style.cursor = 'grabbing';
  }
  
  console.log('Started dragging', dragStartPos.value);
};

const doDrag = (event: MouseEvent) => {
  if (!isDragging.value || !diagramModeActive.value) return;
  
  const dx = event.clientX - dragStartPos.value.x;
  const dy = event.clientY - dragStartPos.value.y;
  
  // Update current translation
  currentTranslate.value = {
    x: currentTranslate.value.x + dx,
    y: currentTranslate.value.y + dy
  };
  
  // Update drag start position for next movement
  dragStartPos.value = { x: event.clientX, y: event.clientY };
  
  // Apply the transform
  applyZoom();
  
  console.log('Dragging', currentTranslate.value);
};

const endDrag = () => {
  if (!isDragging.value) return;
  
  isDragging.value = false;
  
  // Reset cursor
  if (diagramEl.value) {
    diagramEl.value.style.cursor = 'grab';
  }
  
  console.log('Ended dragging');
};

// Handle mouse wheel zoom on diagram
const handleDiagramWheel = (event: WheelEvent) => {
  if (diagramModeActive.value) {
    // Prevent the default scroll behavior
    event.preventDefault();
    event.stopPropagation();
    
    console.log('Wheel event detected', event.deltaY);
    
    if (event.deltaY < 0) {
      // Scrolling up - zoom in
      zoomIn();
    } else {
      // Scrolling down - zoom out
      zoomOut();
    }
  }
};

// Function to set up diagram zoom event listeners
const setupDiagramZoom = () => {
  if (diagramEl.value) {
    // First remove any existing listener to avoid duplicates
    diagramEl.value.removeEventListener('wheel', handleDiagramWheel as EventListener);
    
    // Then add the listener with the passive option set to false to allow preventDefault
    diagramEl.value.addEventListener('wheel', handleDiagramWheel as EventListener, { passive: false });
    
    // Add drag listeners
    diagramEl.value.removeEventListener('mousedown', startDrag);
    diagramEl.value.addEventListener('mousedown', startDrag);
    
    // Add global mouse move and up listeners
    document.removeEventListener('mousemove', doDrag);
    document.addEventListener('mousemove', doDrag);
    
    document.removeEventListener('mouseup', endDrag);
    document.addEventListener('mouseup', endDrag);
    
    // Set initial cursor style
    diagramEl.value.style.cursor = 'grab';
    
    // Also try to attach directly to the SVG once it's available
    setTimeout(() => {
      const svgElement = diagramEl.value?.querySelector('.mermaid svg');
      if (svgElement) {
        svgElement.removeEventListener('wheel', handleDiagramWheel as EventListener);
        svgElement.addEventListener('wheel', handleDiagramWheel as EventListener, { passive: false });
      }
    }, 500);
    
    console.log('Wheel event listeners set up for diagram zooming');
  }
};

// Toggle diagram mode
const toggleDiagramMode = () => {
  // Toggle the mode
  diagramModeActive.value = !diagramModeActive.value;
  
  if (diagramModeActive.value) {
    // Reset zoom and position when entering diagram mode
    diagramZoom.value = 1;
    currentTranslate.value = { x: 0, y: 0 };
    isDragging.value = false;
    
    // Add wheel event listener for zooming and mouse drag for panning
    setTimeout(() => {
      setupDiagramZoom();
    }, 200);
  } else {
    // Remove event listeners when exiting diagram mode
    if (diagramEl.value) {
      diagramEl.value.removeEventListener('wheel', handleDiagramWheel as EventListener);
      diagramEl.value.removeEventListener('mousedown', startDrag);
      document.removeEventListener('mousemove', doDrag);
      document.removeEventListener('mouseup', endDrag);
      console.log('Event listeners removed');
    }
    
    // Exiting diagram mode - ensure editor content is properly restored
    setTimeout(() => {
      // Add a small delay to ensure the DOM has updated
      if (editorEl.value) {
        // Force re-render of the editor if needed
        const currentHtml = editorEl.value.innerHTML;
        if (!currentHtml || currentHtml.trim() === '') {
          // Check if we have a current version loaded
          if (currentVersionIndex.value !== null) {
            // Load the current version content
            const version = promptVersions.value[currentVersionIndex.value];
            if (version) {
              if (selectedLanguage.value === 'pt-BR' && version.html) {
                editorEl.value.innerHTML = version.html;
              } else if (version.languages && 
                      selectedLanguage.value in version.languages && 
                      version.languages[selectedLanguage.value]?.html) {
                editorEl.value.innerHTML = version.languages[selectedLanguage.value]?.html || '';
              } else if (version.html) {
                // Fallback to PT-BR if selected language isn't available
                editorEl.value.innerHTML = version.html;
              } else {
                // Default if no HTML content is available
                editorEl.value.innerHTML = '<p>Comece a escrever seu prompt aqui...</p>';
              }
            } else {
              // Default content if no version is loaded
              editorEl.value.innerHTML = '<p>Comece a escrever seu prompt aqui...</p>';
            }
          } else {
            // Default content if no version is loaded
            editorEl.value.innerHTML = '<p>Comece a escrever seu prompt aqui...</p>';
          }
          updateMarkdown();
        }
      }
    }, 0);
  }
};

// Types for the version system
interface PromptVersion {
  name: string;
  timestamp: string;
  content: string;
  html: string;
  languages?: {
    [langCode: string]: {
      content: string;
      html: string;
    }
  };
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

// Language support
const languageOptions = [
  { label: '🇧🇷 Portuguese (BR)', value: 'pt-BR' },
  { label: '🇵🇹 Portuguese (PT)', value: 'pt-PT' },
  { label: '🇺🇸 English', value: 'en-US' },
  { label: '🇪🇸 Spanish', value: 'es-ES' }
];
const selectedLanguage = ref('pt-BR');
const currentLoadedLanguage = ref('pt-BR');

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
  
  // Get the current version
  const version = promptVersions.value[currentVersionIndex.value];
  if (!version) return false;
  
  // If user has selected a language that doesn't exist yet for this version
  const existingLanguages = getExistingLanguagesForVersion(currentVersionIndex.value);
  if (!existingLanguages.includes(selectedLanguage.value)) {
    return true; // Enable save button to create a new language variant
  }
  
  // Check for content changes in the current language
  if (selectedLanguage.value === 'pt-BR') {
    // For PT-BR, check against the main content
    return markdownOutput.value !== lastSavedContent.value;
  } else if (version.languages) {
    // For other languages, check against the language variant content
    const langVariant = version.languages[selectedLanguage.value];
    if (langVariant && langVariant.content) {
      return markdownOutput.value !== langVariant.content;
    }
  }
  
  return false;
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

// Computed property to determine available language options that haven't been saved yet
const availableLanguageOptions = computed(() => {
  // If no version is selected, show all languages
  if (currentVersionIndex.value === null) {
    return languageOptions;
  }
  
  // Get the existing languages for this version
  const existingLanguages = getExistingLanguagesForVersion(currentVersionIndex.value);
  
  // Always include the currently selected language to avoid UI issues
  // We need to make sure it's visible in the dropdown even if it already exists
  const visibleLanguages = [...existingLanguages];
  if (!visibleLanguages.includes(selectedLanguage.value)) {
    visibleLanguages.push(selectedLanguage.value);
  }
  
  // Return all language options except those already saved (unless it's the current selection)
  return languageOptions.filter(option => 
    !existingLanguages.includes(option.value) || option.value === selectedLanguage.value
  );
});

// Check if the selected language is already saved
const isLanguageAlreadySaved = computed(() => {
  if (currentVersionIndex.value === null) return false;
  
  // Get all languages that exist for the current version
  const existingLanguages = getExistingLanguagesForVersion(currentVersionIndex.value);
  
  // Check if the selected language is in the list of existing languages
  return existingLanguages.includes(selectedLanguage.value);
});

// Get tooltip message for save button based on state
const getSaveButtonTooltip = () => {
  if (!hasUnsavedChanges.value) {
    return 'Make changes or select a new language to enable saving';
  }
  
  const langLabel = getLanguageLabel(selectedLanguage.value);
  
  // Check if we're switching to a different language
  if (selectedLanguage.value !== currentLoadedLanguage.value) {
    if (currentVersionIndex.value === null) {
      return `Save as a new version in ${langLabel}`;
    }
    
    // If the language already exists for this version
    if (isLanguageAlreadySaved.value) {
      return `Update existing ${langLabel} version`;
    }
    
    return `Create new ${langLabel} variant for this version`;
  }
  
  // Normal save with the current language
  if (currentVersionIndex.value !== null) {
    return `Save changes to ${langLabel} version`;
  }
  
  if (allSlotsFilled.value) {
    return 'All slots filled. Delete a version first.';
  }
  
  return `Save as a new version in ${langLabel}`;
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
  try {
    // Skip if we're in IME composition mode (for special characters)
    if (isComposing.value) return;
    
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
      let tempNode = currentNode;
      
      while (tempNode) {
        if (tempNode.nodeName === 'H6') {
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
  } catch (error) {
    console.error('Error handling keydown:', error);
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

// Add a ref to track if we're in IME composition mode
const isComposing = ref(false);

// Handle the start of IME composition (for special characters)
const handleCompositionStart = () => {
  isComposing.value = true;
};

// Handle the end of IME composition (for special characters)
const handleCompositionEnd = () => {
  isComposing.value = false;
  
  // Make sure to update the markdown with the composed text
  updateMarkdown();
};

// Update markdown output
const updateMarkdown = () => {
  // Skip updates during IME composition to prevent breaking special character input
  if (isComposing.value) return;
  
  if (editorEl.value) {
    try {
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
      
      // Update the markdown output from the current HTML content
      const html = editorEl.value.innerHTML;
      markdownOutput.value = turndownService.turndown(html);
    } catch (error) {
      console.error('Error updating markdown:', error);
    }
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
    
    // Switch to versioning tab to show updated data
    activeTab.value = 'versioning';
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
    
    // Switch to versioning tab to show updated data
    activeTab.value = 'versioning';
  }
};

// Update the current version with the latest content
const updateCurrentVersion = () => {
  if (!editorEl.value || currentVersionIndex.value === null) return;
  
  const version = promptVersions.value[currentVersionIndex.value];
  if (!version) return;
  
  // If we're updating the default language
  if (selectedLanguage.value === 'pt-BR') {
    // Update the version content
    version.content = markdownOutput.value;
    version.html = editorEl.value.innerHTML;
    
    // Update saved content reference
    lastSavedContent.value = markdownOutput.value;
    
    // Make sure both currentLoadedLanguage and selectedLanguage are in sync
    currentLoadedLanguage.value = 'pt-BR';
    
    $q.notify({
      color: 'positive',
      message: `Updated ${version.name} (${getLanguageLabel('pt-BR')})`,
      icon: 'save'
    });
  } else {
    // We're updating a language variant
    if (!version.languages) {
      version.languages = {};
    }
    
    // Update or create the language variant
    version.languages[selectedLanguage.value] = {
      content: markdownOutput.value,
      html: editorEl.value.innerHTML
    };
    
    // Update saved content reference
    lastSavedContent.value = markdownOutput.value;
    
    // Make sure the currentLoadedLanguage matches selectedLanguage
    currentLoadedLanguage.value = selectedLanguage.value;
    
    $q.notify({
      color: 'positive',
      message: `Updated ${version.name} (${getLanguageLabel(selectedLanguage.value)})`,
      icon: 'save'
    });
  }
};

// Create a new version in the specified slot
const createNewVersion = (index: number) => {
  if (!editorEl.value) return;
  
  // Get the current selected language (default to pt-BR if not set)
  const chosenLanguage = selectedLanguage.value || 'pt-BR';
  
  // Create version object with basic information
  const version: PromptVersion = {
    name: getDefaultVersionName(index),
    timestamp: createTimestamp(),
    content: '',
    html: '',
    languages: {}
  };
  
  // Save content in the appropriate place based on selected language
  if (chosenLanguage === 'pt-BR') {
    // Store directly in the version's main content fields
    version.content = markdownOutput.value;
    version.html = editorEl.value.innerHTML;
  } else {
    // Store in the language variant
    if (!version.languages) {
      version.languages = {};
    }
    version.languages[chosenLanguage] = {
      content: markdownOutput.value,
      html: editorEl.value.innerHTML
    };
  }
  
  console.log('Creating new version with language:', chosenLanguage);
  
  promptVersions.value[index] = version;
  currentVersionIndex.value = index;
  lastSavedContent.value = markdownOutput.value;
  currentLoadedLanguage.value = chosenLanguage;
  
  $q.notify({
    color: 'positive',
    message: `Saved as ${version.name} (${getLanguageLabel(chosenLanguage)})`,
    icon: 'save'
  });
};

// Load a version from the version list
const loadVersion = (index: number) => {
  const version = promptVersions.value[index];
  if (!version || !editorEl.value) return;
  
  // Get the list of available languages for this version
  const availableLanguages = getExistingLanguagesForVersion(index);
  
  if (availableLanguages.length === 0) {
    // This should never happen with proper version creation
    console.error('Error: No content available for this version in any language!');
    $q.notify({
      color: 'negative',
      message: 'Error: No content available for this version!',
      icon: 'error'
    });
    return;
  }
  
  // Prioritize pt-BR if it exists, otherwise use the first available language
  // We know availableLanguages is non-empty at this point
  const langToLoad = availableLanguages.includes('pt-BR') ? 'pt-BR' : availableLanguages[0];
  
  // TypeScript assertion to assure it's a valid string
  loadLanguageVariant(index, langToLoad as string);
  
  // Switch back to markdown tab when loading a version
  activeTab.value = 'markdown';
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
          // Check if we have a current version loaded
          if (currentVersionIndex.value !== null) {
            // Load the current version content
            const version = promptVersions.value[currentVersionIndex.value];
            if (version) {
              if (selectedLanguage.value === 'pt-BR' && version.html) {
                editorEl.value.innerHTML = version.html;
              } else if (version.languages && 
                      selectedLanguage.value in version.languages && 
                      version.languages[selectedLanguage.value]?.html) {
                editorEl.value.innerHTML = version.languages[selectedLanguage.value]?.html || '';
              } else if (version.html) {
                // Fallback to PT-BR if selected language isn't available
                editorEl.value.innerHTML = version.html;
              } else {
                // Default if no HTML content is available
                editorEl.value.innerHTML = '<p>Comece a escrever seu prompt aqui...</p>';
              }
            } else {
              // Default content if no version is loaded
              editorEl.value.innerHTML = '<p>Comece a escrever seu prompt aqui...</p>';
            }
          } else {
            // Default content if no version is loaded
            editorEl.value.innerHTML = '<p>Comece a escrever seu prompt aqui...</p>';
          }
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
    // Get API config from env vars
    const apiUrl = import.meta.env.VITE_API_URL || 'http://localhost:3008/v1/platformagents/createPromptEditor';
    const apiToken = getApiToken();

    console.log('API URL:', apiUrl);
    console.log('API Token preview:', apiToken ? `${apiToken.substring(0, 10)}...` : 'No token found');
    
    // Make the API request
    const response = await fetch(apiUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${apiToken}`
      },
      body: JSON.stringify({
        agent: "promptEditor",
        payload: {
          prompt: `This is my current prompt:\n${markdownOutput.value}\n\nThis is my instructions:\n${aiPromptInput.value}`
        }
      })
    });
    
    if (!response.ok) {
      const errorBody = await response.text().catch(() => 'Could not read error response');
      console.error('API Error details:', {
        status: response.status,
        statusText: response.statusText,
        headers: Object.fromEntries([...response.headers.entries()]),
        body: errorBody
      });
      throw new Error(`API error: ${response.status} - ${response.statusText}`);
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
  
  // Process the markdown to handle special bullet points with asterisks and numbers
  // First, normalize line endings
  const normalizedText = markdownText.replace(/\r\n/g, '\n');
  
  // Create an array to build the HTML content line by line
  const lines = normalizedText.split('\n');
  const htmlLines: string[] = [];
  let inList = false;
  
  for (let i = 0; i < lines.length; i++) {
    let line = lines[i] || '';
    
    // Process headers
    if (/^#{1,6}\s+(.+)$/.test(line)) {
      if (inList) {
        htmlLines.push('</ul>');
        inList = false;
      }
      htmlLines.push(line.replace(/^#{1,6}\s+(.+)$/, '<h6>$1</h6>'));
      continue;
    }
    
    // Process complex bullet points with numbers and asterisks
    // Match patterns like: \* 1. Text or * 1. Text or 1. Text or just * Text
    if (/^\s*(\\?\*\s*\d+\.|\\?\*|[-+]|\d+\.)\s+(.+)$/.test(line)) {
      if (!inList) {
        htmlLines.push('<ul>');
        inList = true;
      }
      
      // Extract the content part from the bullet
      const content = line.replace(/^\s*(\\?\*\s*\d+\.|\\?\*|[-+]|\d+\.)\s+(.+)$/, '$2');
      htmlLines.push(`<li>${content}</li>`);
      continue;
    }
    
    // Close list if the next line isn't a bullet point
    if (inList && !/^\s*(\\?\*\s*\d+\.|\\?\*|[-+]|\d+\.)\s+/.test(line)) {
      htmlLines.push('</ul>');
      inList = false;
    }
    
    // Process bold text
    line = line.replace(/\*\*(.+?)\*\*/g, '<strong>$1</strong>');
    
    // Process italic text
    line = line.replace(/\*(.+?)\*/g, '<em>$1</em>');
    
    // If not a special line, treat as regular paragraph
    if (line.trim()) {
      htmlLines.push(`<p>${line}</p>`);
    }
  }
  
  // Close any open list at the end
  if (inList) {
    htmlLines.push('</ul>');
  }
  
  // Join the processed lines to form the HTML content
  const htmlContent = htmlLines.join('');
  
  // Store the HTML content to be applied after AI mode is exited
  // We need to use a short timeout to ensure the DOM has updated after exiting AI mode
  setTimeout(() => {
    if (!editorEl.value) return;
    
    try {
      // Use sanitizeHtml to clean the content before applying
      const sanitizedHtml = sanitizeHtml(htmlContent);
      
      // Remember the current scroll position
      const scrollPos = editorEl.value.scrollTop;
      
      // Set the content to the editor element
      editorEl.value.innerHTML = sanitizedHtml;
      
      // Restore scroll position
      editorEl.value.scrollTop = scrollPos;
      
      // Manually update the markdown output to reflect the AI-generated content
      // But don't run any post-processing to avoid breaking special characters
      markdownOutput.value = turndownService.turndown(sanitizedHtml);
      
      console.log('AI content applied:', htmlContent);
      console.log('Markdown updated:', markdownOutput.value);
    } catch (error) {
      console.error('Error applying AI content:', error);
      // Fallback to simple insertion
      editorEl.value.innerHTML = htmlContent;
      markdownOutput.value = turndownService.turndown(htmlContent);
    }
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
  
  // Keep the current language selection
  // This allows users to create new prompts in their preferred language
  currentLoadedLanguage.value = selectedLanguage.value;
  
  $q.notify({
    color: 'info',
    message: `Created new blank prompt (${getLanguageLabel(selectedLanguage.value)})`,
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
  const hasMarkdownSyntax = /#{1,6}\s|[*_]|^\s*[-+*]|\\?\*|^\s*\d+\.|`{1,3}|>\s/.test(pastedText);
  
  if (hasMarkdownSyntax) {
    // Try to convert markdown to HTML
    try {
      // Normalize line endings
      const normalizedText = pastedText.replace(/\r\n/g, '\n');
      
      // Process line by line for more complex bullet point handling
      const lines = normalizedText.split('\n');
      const htmlLines: string[] = [];
      let inList = false;
      
      for (let i = 0; i < lines.length; i++) {
        let line = lines[i] || '';
        
        // Process headings (h6 for our app)
        if (/^#{1,6}\s+(.+)$/.test(line)) {
          if (inList) {
            htmlLines.push('</ul>');
            inList = false;
          }
          htmlLines.push(line.replace(/^#{1,6}\s+(.+)$/, '<h6>$1</h6>'));
          continue;
        }
        
        // Process complex bullet points with numbers and asterisks
        // Match patterns like: \* 1. Text or * 1. Text or 1. Text or just * Text
        if (/^\s*(\\?\*\s*\d+\.|\\?\*|[-+]|\d+\.)\s+(.+)$/.test(line)) {
          if (!inList) {
            htmlLines.push('<ul>');
            inList = true;
          }
          
          // Extract the content part from the bullet
          const content = line.replace(/^\s*(\\?\*\s*\d+\.|\\?\*|[-+]|\d+\.)\s+(.+)$/, '$2');
          htmlLines.push(`<li>${content}</li>`);
          continue;
        }
        
        // Close list if the next line isn't a bullet point
        if (inList && !/^\s*(\\?\*\s*\d+\.|\\?\*|[-+]|\d+\.)\s+/.test(line)) {
          htmlLines.push('</ul>');
          inList = false;
        }
        
        // Process bold text
        line = line.replace(/\*\*(.+?)\*\*/g, '<strong>$1</strong>').replace(/__(.+?)__/g, '<strong>$1</strong>');
        
        // Process italic text
        line = line.replace(/\*(.+?)\*/g, '<em>$1</em>').replace(/_(.+?)_/g, '<em>$1</em>');
        
        // Check for color syntax
        line = line.replace(/{color:([\w#]+)}(.+?){\/color}/g, '<span style="color:$1">$2</span>');
        
        // If not a special line and not empty, treat as paragraph
        if (line.trim() && !line.includes('<h6>') && !line.includes('<li>')) {
          htmlLines.push(`<p>${line}</p>`);
        } else if (!line.trim()) {
          // Add empty line if needed
          htmlLines.push('<p><br></p>');
        }
      }
      
      // Close any open list at the end
      if (inList) {
        htmlLines.push('</ul>');
      }
      
      // Join all lines into final HTML
      const processedHtml = htmlLines.join('');
      
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

// Helper function to get language label from code
const getLanguageLabel = (langCode: string): string => {
  const option = languageOptions.find(opt => opt.value === langCode);
  return option ? option.label : langCode;
};

// Helper to get just the flag emoji for a language
const getFlagForLanguage = (langCode: string): string => {
  switch (langCode) {
    case 'pt-BR': return '🇧🇷';
    case 'pt-PT': return '🇵🇹';
    case 'en-US': return '🇺🇸';
    case 'es-ES': return '🇪🇸';
    default: return '🌐';
  }
};

// Load a specific language variant for a version
const loadLanguageVariant = (index: number, langCode: string) => {
  const version = promptVersions.value[index];
  if (!version || !editorEl.value) return;
  
  let contentLoaded = false;
  
  // Try to load the requested language
  if (langCode === 'pt-BR' && version.content && version.html) {
    // Load the pt-BR content from the main version fields
    editorEl.value.innerHTML = version.html;
    lastSavedContent.value = version.content;
    selectedLanguage.value = 'pt-BR';
    currentLoadedLanguage.value = 'pt-BR';
    contentLoaded = true;
  } else if (version.languages) {
    // Check if the requested language variant exists with content
    const langVariant = version.languages[langCode];
    if (langVariant && langVariant.html && langVariant.content) {
      editorEl.value.innerHTML = langVariant.html;
      lastSavedContent.value = langVariant.content;
      selectedLanguage.value = langCode;
      currentLoadedLanguage.value = langCode;
      contentLoaded = true;
    } else {
      // Try pt-BR as fallback if we have it
      if (version.content && version.html) {
        editorEl.value.innerHTML = version.html;
        lastSavedContent.value = version.content;
        selectedLanguage.value = 'pt-BR';
        currentLoadedLanguage.value = 'pt-BR';
        
        $q.notify({
          color: 'warning',
          message: `Language ${getLanguageLabel(langCode)} not available. Loaded ${version.name} in Portuguese (BR).`,
          icon: 'warning'
        });
        contentLoaded = true;
      } else {
        // Try any available language
        const availableLangs = getExistingLanguagesForVersion(index);
        if (availableLangs.length > 0 && availableLangs[0] !== langCode) {
          // TypeScript assertion since we already checked the array has items
          const otherLang = availableLangs[0] as string;
          
          // We already know this language exists in the version
          if (otherLang === 'pt-BR' && version.content && version.html) {
            // Load PT-BR content
            editorEl.value.innerHTML = version.html;
            lastSavedContent.value = version.content;
          } else if (version.languages && version.languages[otherLang]) {
            // We'll check again to ensure TypeScript is happy
            if (otherLang && version.languages[otherLang]) {
              // Load other language content (already validated by getExistingLanguagesForVersion)
              const otherLangVariant = version.languages[otherLang];
              if (otherLangVariant && otherLangVariant.html && otherLangVariant.content) {
                editorEl.value.innerHTML = otherLangVariant.html;
                lastSavedContent.value = otherLangVariant.content;
              } else {
                // This should never happen due to validation in getExistingLanguagesForVersion
                contentLoaded = false;
                console.error('Error: Language exists but content is invalid');
                return;
              }
            }
          }
          
          selectedLanguage.value = otherLang;
          currentLoadedLanguage.value = otherLang;
          
          $q.notify({
            color: 'warning',
            message: `Language ${getLanguageLabel(langCode)} not available. Loaded ${version.name} in ${getLanguageLabel(otherLang)}.`,
            icon: 'warning'
          });
          contentLoaded = true;
        }
      }
    }
  }
  
  // Update state if we loaded content
  if (contentLoaded) {
    updateMarkdown();
    currentVersionIndex.value = index;
    
    $q.notify({
      color: 'info',
      message: `Loaded ${version.name} (${getLanguageLabel(currentLoadedLanguage.value)})`,
      icon: 'history'
    });
  } else {
    // This should never happen with proper version creation
    console.error('Error: No content available for this version in any language!');
    $q.notify({
      color: 'negative',
      message: 'Error: No content available for this version!',
      icon: 'error'
    });
  }
};

// Delete a language variant
const deleteLanguageVariant = (index: number, langCode: string) => {
  // Don't allow deleting the default language (pt-BR)
  if (langCode === 'pt-BR') {
    $q.notify({
      color: 'negative',
      message: 'Cannot delete the default language',
      icon: 'error'
    });
    return;
  }
  
  const version = promptVersions.value[index];
  if (!version || !version.languages) return;
  
  // Ask for confirmation
  $q.dialog({
    title: 'Confirm Deletion',
    message: `Are you sure you want to delete the ${getLanguageLabel(langCode)} variant?`,
    cancel: true,
    persistent: true
  }).onOk(() => {
    // Delete the language variant
    if (version.languages) {
      delete version.languages[langCode];
      
      // If this was the currently loaded language, reset to default
      if (currentVersionIndex.value === index && currentLoadedLanguage.value === langCode) {
        loadLanguageVariant(index, 'pt-BR');
      }
      
      $q.notify({
        color: 'negative',
        message: `Deleted ${getLanguageLabel(langCode)} variant`,
        icon: 'delete'
      });
    }
  });
};

// Handle language change
const handleLanguageChange = () => {
  // For unsaved versions (no version selected yet)
  if (currentVersionIndex.value === null) {
    // Just update the currentLoadedLanguage
    currentLoadedLanguage.value = selectedLanguage.value;
    
    $q.notify({
      color: 'info',
      message: `Switched to ${getLanguageLabel(selectedLanguage.value)}. Save to create version in this language.`,
      icon: 'language'
    });
    return;
  }
  
  // For saved versions:
  const versionIndex = currentVersionIndex.value;
  const version = promptVersions.value[versionIndex];
  if (!version) return;
  
  // Check if the selected language already exists for this version
  const existingLanguages = getExistingLanguagesForVersion(versionIndex);
  const languageExists = existingLanguages.includes(selectedLanguage.value);
  
  // If we have unsaved changes and switching to a different language
  if (hasUnsavedChanges.value && selectedLanguage.value !== currentLoadedLanguage.value) {
    // If the target language already exists, ask if user wants to load it or overwrite
    if (languageExists) {
      $q.dialog({
        title: 'Language Variant Exists',
        message: `A ${getLanguageLabel(selectedLanguage.value)} variant already exists. Do you want to load it (replacing current content) or keep your current edits to save in this language?`,
        options: {
          type: 'radio',
          model: 'load',
          items: [
            { label: 'Load existing content', value: 'load' },
            { label: 'Keep current changes', value: 'keep' }
          ]
        },
        persistent: true
      }).onOk(action => {
        if (action === 'load') {
          // User wants to load the existing language variant
          loadLanguageVariant(versionIndex, selectedLanguage.value);
        } else {
          // User wants to keep current changes to save in the new language
          // Just update the currentLoadedLanguage to match the selectedLanguage
          currentLoadedLanguage.value = selectedLanguage.value;
          
          // Update lastSavedContent to the content of the selected language
          // so hasUnsavedChanges will return true
          if (selectedLanguage.value === 'pt-BR' && version.content) {
            lastSavedContent.value = version.content;
          } else if (version.languages && version.languages[selectedLanguage.value]) {
            const langVariant = version.languages[selectedLanguage.value];
            if (langVariant && langVariant.content) {
              lastSavedContent.value = langVariant.content;
            }
          }
          
          $q.notify({
            color: 'info',
            message: `Switched to ${getLanguageLabel(selectedLanguage.value)}. Save to create/update this language variant.`,
            icon: 'language'
          });
        }
      }).onCancel(() => {
        // Revert the selection
        selectedLanguage.value = currentLoadedLanguage.value;
      });
      return;
    }
    
    // If we're here, the target language doesn't exist yet
    // Just update the currentLoadedLanguage
    currentLoadedLanguage.value = selectedLanguage.value;
    
    $q.notify({
      color: 'info',
      message: `Switched to ${getLanguageLabel(selectedLanguage.value)}. Save to create/update this language variant.`,
      icon: 'language'
    });
    return;
  }
  
  // If no unsaved changes or switching to the same language with changes
  if (!hasUnsavedChanges.value || selectedLanguage.value === currentLoadedLanguage.value) {
    // We can safely load the selected language variant if it exists
    if (languageExists) {
      loadLanguageVariant(versionIndex, selectedLanguage.value);
    } else {
      // Language doesn't exist yet, just update the currentLoadedLanguage
      currentLoadedLanguage.value = selectedLanguage.value;
      // This will enable the save button (because hasUnsavedChanges will return true)
      $q.notify({
        color: 'info',
        message: `Switched to ${getLanguageLabel(selectedLanguage.value)}. Edit and save to create this language variant.`,
        icon: 'language'
      });
    }
  }
};

// Helper to get all languages available for a specific version
const getExistingLanguagesForVersion = (versionIndex: number | null): string[] => {
  if (versionIndex === null) return [];
  
  const version = promptVersions.value[versionIndex];
  if (!version) return [];
  
  const existingLanguages: string[] = [];
  
  // Add PT-BR if it has content
  if (version.content && version.content.trim() !== '') {
    existingLanguages.push('pt-BR');
  }
  
  // Add other language variants
  if (version.languages) {
    Object.keys(version.languages).forEach(lang => {
      // Only add languages that have valid content
      const langVariant = version.languages?.[lang];
      if (langVariant && langVariant.content && langVariant.content.trim() !== '' && !existingLanguages.includes(lang)) {
        existingLanguages.push(lang);
      }
    });
  }
  
  return existingLanguages;
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

// Computed property to format version data as JSON
const formattedVersionData = computed(() => {
  // Create a copy of the data we want to display
  const versionData = {
    editorState: {
      currentVersionIndex: currentVersionIndex.value,
      selectedLanguage: selectedLanguage.value,
      currentLoadedLanguage: currentLoadedLanguage.value,
      hasUnsavedChanges: hasUnsavedChanges.value
    },
    // Filter out null entries and only include non-null versions
    versions: promptVersions.value
      .map((version, index) => {
        if (!version) return null;
        
        // Get available languages for this version
        const availableLanguages = getExistingLanguagesForVersion(index);
        
        // Build language content map with the actual prompt content
        const languageContents: Record<string, {
          content: string;
          isCurrentlyLoaded: boolean;
        }> = {};
        
        // Add the default language (pt-BR) content
        if (availableLanguages.includes('pt-BR')) {
          languageContents['pt-BR'] = {
            content: showFullPromptContent.value ? version.content : (version.content.length > 50 ? version.content.substring(0, 50) + '...' : version.content),
            isCurrentlyLoaded: currentVersionIndex.value === index && currentLoadedLanguage.value === 'pt-BR'
          };
        }
        
        // Add other languages
        if (version.languages) {
          Object.keys(version.languages).forEach(lang => {
            if (version.languages && version.languages[lang] && version.languages[lang].content) {
              const content = version.languages[lang].content;
              languageContents[lang] = {
                content: showFullPromptContent.value ? content : (content.length > 50 ? content.substring(0, 50) + '...' : content),
                isCurrentlyLoaded: currentVersionIndex.value === index && currentLoadedLanguage.value === lang
              };
            }
          });
        }
        
        // Format version data for display
        return {
          versionIndex: index,
          name: version.name,
          timestamp: version.timestamp,
          isCurrent: index === currentVersionIndex.value,
          availableLanguages,
          // Include the actual content for each language
          promptContent: languageContents
        };
      })
      .filter(v => v !== null) // Remove null entries
  };
  
  // Format as pretty JSON with 2-space indentation
  return JSON.stringify(versionData, null, 2);
});

// Refresh version data
const refreshVersionData = () => {
  // Since our data is reactive, we just need to create an update that triggers reactivity
  // This will make the formattedVersionData computed property re-evaluate
  
  // Create a temporary ref that we update to force reactivity
  const tempRef = ref(0);
  tempRef.value++;
  
  // Show notification to confirm refresh
  $q.notify({
    color: 'info',
    message: 'Version data refreshed',
    icon: 'refresh',
    timeout: 1000
  });
};

// Download version data as JSON
const downloadVersionData = () => {
  // Create a Blob with the JSON data
  const blob = new Blob([formattedVersionData.value], { type: 'application/json' });
  
  // Create a temporary anchor element to trigger download
  const link = document.createElement('a');
  link.href = URL.createObjectURL(blob);
  link.download = `prompt-versions-${Date.now()}.json`;
  link.click();
  
  // Clean up
  URL.revokeObjectURL(link.href);
  
  // Show notification
  $q.notify({
    color: 'positive',
    message: 'Prompt versions exported as JSON',
    icon: 'file_download',
    timeout: 2000
  });
};

// Computed property to provide syntax-highlighted version data
const highlightedVersionData = computed(() => {
  // Start with the formatted JSON
  let json = formattedVersionData.value;
  
  // Replace with syntax highlighting
  // First escape HTML
  json = json.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;');
  
  // Add syntax highlighting with regex replacements
  // Highlight strings
  json = json.replace(/"(\\u[a-zA-Z0-9]{4}|\\[^u]|[^\\"])*"(\s*:)?/g, match => {
    const isKey = /:$/.test(match);
    return '<span class="' + (isKey ? 'key' : 'string') + '">' + match + '</span>';
  });
  
  // Highlight numbers
  json = json.replace(/\b(\d+\.\d+|\d+)\b/g, '<span class="number">$1</span>');
  
  // Highlight booleans
  json = json.replace(/\b(true|false)\b/g, '<span class="boolean">$1</span>');
  
  // Highlight null
  json = json.replace(/\bnull\b/g, '<span class="null">null</span>');
  
  return json;
});

// AI Diagram Generator
const submitDiagramPrompt = async () => {
  // If we're already in diagram mode, toggle back to editor
  if (diagramModeActive.value) {
    diagramModeActive.value = false;
    return;
  }

  isProcessingDiagramRequest.value = true;
  
  try {
    // Get API config from env vars
    const apiUrl = import.meta.env.VITE_API_URL_DIAGRAM || 'http://localhost:3008/v1/platformagents/createDiagram';
    const apiToken = import.meta.env.VITE_API_TOKEN;
    
    console.log('Sending diagram request to:', apiUrl);
    console.log('API Token preview:', apiToken ? `${apiToken.substring(0, 10)}...` : 'No token found');
    
    // Make the API request
    const response = await fetch(apiUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${apiToken}`
      },
      body: JSON.stringify({
        agent: "diagram",
        payload: {
          prompt: markdownOutput.value
        }
      })
    });
    
    if (!response.ok) {
      const errorBody = await response.text().catch(() => 'Could not read error response');
      console.error('API Error details:', {
        status: response.status,
        statusText: response.statusText,
        headers: Object.fromEntries([...response.headers.entries()]),
        body: errorBody
      });
      throw new Error(`API error: ${response.status} - ${response.statusText}`);
    }
    console.log('Diagram generation response:', response);
    
    // Read the response as text instead of trying to parse as JSON
    const diagramText = await response.text();
    console.log('Diagram generation response:', diagramText);
    
    // Toggle diagram mode to show the diagram
    diagramModeActive.value = true;
    
    // Wait for the DOM to update
    setTimeout(() => {
      // Then render the response diagram
      if (diagramText) {
        if (diagramEl.value) {
          // Clear previous content
          diagramEl.value.innerHTML = '';
          
          // Create a div for mermaid
          const mermaidDiv = document.createElement('div');
          mermaidDiv.className = 'mermaid';
          mermaidDiv.textContent = diagramText;
          
          // Add to DOM
          diagramEl.value.appendChild(mermaidDiv);
          
          // Render the diagram
          mermaid.run({ nodes: [mermaidDiv] }).catch(error => {
            console.error('Error rendering mermaid diagram:', error);
          }).finally(() => {
            // Apply initial zoom after diagram is rendered and setup interaction
            setTimeout(() => {
              // Reset the position and zoom
              currentTranslate.value = { x: 0, y: 0 };
              diagramZoom.value = 1;
              
              applyZoom();
              setupDiagramZoom();
            }, 100);
          });
        }
        
      } else {
        throw new Error('Invalid API response format');
      }
    }, 100);
  } catch (error) {
    console.error('Error calling AI Diagram API:', error);
    
    $q.notify({
      color: 'negative',
      message: `Error generating AI diagram: ${error instanceof Error ? error.message : 'Unknown error'}`,
      icon: 'error',
      timeout: 5000,
      position: 'top'
    });
  } finally {
    // Reset the processing state
    isProcessingDiagramRequest.value = false;
  }
};
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

/* Language selector styles */
.language-selector :deep(.q-field__control) {
  height: 32px;
  min-height: unset;
  padding: 0 8px;
}

.language-selector :deep(.q-field__marginal) {
  height: 32px;
  margin-left: 0;
}

.language-selector :deep(.q-field__native) {
  padding: 0;
  display: flex;
  justify-content: center;
}

.language-selector :deep(.q-field__control-container) {
  padding-right: 0;
}

.flag-icon {
  display: inline-block;
  font-size: 20px;
  line-height: 1;
  margin-right: 0;
}

/* Language menu styling */
.language-menu {
  max-height: 400px;
  overflow-y: auto;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.15);
  border-radius: 4px;
  min-width: 200px;
  border-left: 1px solid #e0e0e0;
  margin-left: -1px;
}

.submenu-list {
  padding: 8px 0;
}

.submenu-header {
  font-size: 14px;
  font-weight: 500;
  padding: 8px 16px;
  margin: 0;
}

.language-menu :deep(.q-item) {
  min-height: 32px;
  padding: 4px 16px;
}

.language-menu :deep(.q-separator--spaced) {
  margin: 8px 0;
}

.bg-primary-1 {
  background-color: #EBE8FD !important;
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
  padding: 0;
  background-color: #f8f9fa;
}

.markdown-content {
  padding: 16px;
  height: 500px;
  overflow-y: auto;
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
  margin-top: 6px;
  margin-bottom: 6px;
}

.editor li {
  margin-bottom: 4px;
  list-style-position: outside;
  display: list-item;
}

/* Improve nested list styling */
.editor ul ul {
  margin-top: 4px;
  margin-bottom: 4px;
}

/* Make sure bullet points are properly displayed */
.editor ul li {
  list-style-type: disc;
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

/* Version selector styling */
.version-selector :deep(.q-btn-dropdown__arrow) {
  margin-left: 4px;
}

/* Ensure action buttons in version menu are horizontally aligned */
.q-item__section--side > .row {
  align-items: center;
  justify-content: flex-end;
}

/* Make sure the right arrow icon is properly visible */
.q-item__section--side .q-btn {
  margin-left: 2px;
}

/* Tab styling */
.tab-button {
  width: 100%;
  justify-content: flex-start;
  padding: 12px 16px;
  transition: all 0.2s ease;
  border-radius: 0;
}

.tab-button:hover {
  background-color: #f0f0f0;
}

.active-tab {
  background-color: #EBE8FD;
  font-weight: bold;
  border-bottom: 2px solid #6467F2;
}

/* Version data display styling */
.version-data-display {
  background-color: #f8f9fa;
  border-radius: 4px;
  border: 1px solid #e0e0e0;
  font-family: 'Fira Code', 'Courier New', monospace;
  font-size: 13px;
  height: 100%;
  overflow-y: auto;
  white-space: pre-wrap;
  word-break: break-word;
  line-height: 1.5;
  max-height: calc(500px - 40px);
  color: #333;
}

.version-data-display .string { color: #008000; }
.version-data-display .number { color: #0000ff; }
.version-data-display .boolean { color: #b22222; }
.version-data-display .null { color: #808080; }
.version-data-display .key { color: #a52a2a; }

/* Diagram container styles */
.diagram-container {
  height: 100%;
  padding: 16px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
}

.diagram-container .text-subtitle1 {
  font-weight: 500;
  color: #6467F2;
}

.diagram-content {
  flex-grow: 1;
  background-color: white;
  border-radius: 8px;
  border: 1px solid #e0e0e0;
  padding: 16px;
  overflow: auto;
  min-height: 350px;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

/* Zoom controls styling */
.diagram-zoom-controls {
  display: flex;
  gap: 4px;
  background-color: rgba(255, 255, 255, 0.7);
  border-radius: 4px;
  padding: 2px 6px;
  z-index: 10;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  align-items: center;
}

.zoom-level {
  font-size: 12px;
  color: #666;
  margin-right: 4px;
  min-width: 40px;
  text-align: center;
}

/* Mermaid diagram styling */
:deep(.mermaid) {
  width: 100%;
  text-align: center;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  overflow: hidden;
}

:deep(.mermaid svg) {
  max-width: 100%;
  height: auto;
  transition: transform 0.2s ease;
  transform-origin: center center;
}
</style> 