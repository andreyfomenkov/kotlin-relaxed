Build

1. Download Kotlin sources from the desired branch to /kotlin
2. Add `kotlin.build.isObsoleteJdkOverrideEnabled=true` to local.properties file in /kotlin directory
3. Apply patch:

diff --git a/compiler/frontend/src/org/jetbrains/kotlin/resolve/calls/smartcasts/DataFlowValueKindUtils.kt b/compiler/frontend/src/org/jetbrains/kotlin/resolve/calls/smartcasts/DataFlowValueKindUtils.kt
index e1d895a7a7b..ff836f5d85c 100644
--- a/compiler/frontend/src/org/jetbrains/kotlin/resolve/calls/smartcasts/DataFlowValueKindUtils.kt
+++ b/compiler/frontend/src/org/jetbrains/kotlin/resolve/calls/smartcasts/DataFlowValueKindUtils.kt
@@ -23,32 +23,6 @@ internal fun PropertyDescriptor.propertyKind(
     usageModule: ModuleDescriptor?,
     languageVersionSettings: LanguageVersionSettings
 ): DataFlowValue.Kind {
-    if (isVar) return DataFlowValue.Kind.MUTABLE_PROPERTY
-    if (isOverridable) return DataFlowValue.Kind.PROPERTY_WITH_GETTER
-    if (!hasDefaultGetter()) return DataFlowValue.Kind.PROPERTY_WITH_GETTER
-    val originalDescriptor = DescriptorUtils.unwrapFakeOverride(this)
-    val isInvisibleFromOtherModuleUnwrappedFakeOverride = originalDescriptor.isInvisibleFromOtherModules()
-    if (isInvisibleFromOtherModuleUnwrappedFakeOverride) return DataFlowValue.Kind.STABLE_VALUE
-
-    if (kind == CallableMemberDescriptor.Kind.FAKE_OVERRIDE) {
-        if (overriddenDescriptors.any { isDeclaredInAnotherModule(usageModule) }) {
-            val deprecationForInvisibleFakeOverride =
-                isInvisibleFromOtherModules() != isInvisibleFromOtherModuleUnwrappedFakeOverride &&
-                        !languageVersionSettings.supportsFeature(ProhibitSmartcastsOnPropertyFromAlienBaseClassInheritedInInvisibleClass)
-            return when {
-                deprecationForInvisibleFakeOverride ->
-                    DataFlowValue.Kind.LEGACY_ALIEN_BASE_PROPERTY_INHERITED_IN_INVISIBLE_CLASS
-                !languageVersionSettings.supportsFeature(ProhibitSmartcastsOnPropertyFromAlienBaseClass) ->
-                    DataFlowValue.Kind.LEGACY_ALIEN_BASE_PROPERTY
-                else ->
-                    DataFlowValue.Kind.ALIEN_PUBLIC_PROPERTY
-            }
-        }
-    }
-
-    val declarationModule = DescriptorUtils.getContainingModule(originalDescriptor)
-    if (!areCompiledTogether(usageModule, declarationModule)) return DataFlowValue.Kind.ALIEN_PUBLIC_PROPERTY
-
     return DataFlowValue.Kind.STABLE_VALUE
 }

diff --git a/gradle.properties b/gradle.properties
index 25913f3bb48..5bd453a6b62 100644
--- a/gradle.properties
+++ b/gradle.properties
@@ -83,7 +83,7 @@ org.gradle.vfs.watch=true
 #kotlin.build.isObsoleteJdkOverrideEnabled=true

 # Disable -Werror compiler flag
-#kotlin.build.disable.werror=true
+kotlin.build.disable.werror=true

 # Render internal diagostic names
 #kotlin.build.render.diagnostic.names=true

 4. Build ./gradlew dist
 5. Create zip for dist/kotlinc and move archive to kotlin-relaxed repo /artifact direcotry
 6. Update /artifact/date file