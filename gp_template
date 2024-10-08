bl_info = {
    "name": "Grease Pencil Blank with Template",
    "author": "Juan Ignacio Fulponi",
    "blender": (3, 0, 0),
    "category": "Object",
    "location": "View3D > Add > Grease Pencil",
    "description": "Create a Grease Pencil template with multiple elements and shadows",
}

import bpy

class GPBlankTemplateOperator(bpy.types.Operator):
    bl_idname = "gpencil.blank_with_template"
    bl_label = "Blank with Template"
    bl_options = {'REGISTER', 'UNDO'}

    num_elements: bpy.props.IntProperty(name="Number of Elements (k)", default=1, min=1)
    separate_objects: bpy.props.BoolProperty(name="Each Element as Separate Object", default=False)
    num_shadows: bpy.props.IntProperty(name="Number of Shadows (n)", default=1, min=0)
    mask_shadows: bpy.props.BoolProperty(name="Mask Shadows", default=True)

    def execute(self, context):
        self.create_materials()

        for i in range(1, self.num_elements + 1):
            if self.separate_objects:
                gp_object = self.create_grease_pencil_object(context, i)
            else:
                if i == 1:
                    gp_object = self.create_grease_pencil_object(context, "Main")

            self.create_element_layers(gp_object, i)

        return {'FINISHED'}

    def create_grease_pencil_object(self, context, name_suffix):
        gp_data = bpy.data.grease_pencils.new(name=f"GPencil_{name_suffix}")
        gp_object = bpy.data.objects.new(name=f"GPencil_Object_{name_suffix}", object_data=gp_data)
        context.collection.objects.link(gp_object)
        context.view_layer.objects.active = gp_object

        # Add Noise modifier
        noise_mod = gp_object.grease_pencil_modifiers.new(name="Noise", type='GP_NOISE')
        noise_mod.factor = 0.4
        noise_mod.noise_scale = 0.2
        noise_mod.step = 2
        noise_mod.show_viewport = False  # Disable for viewport but enable for render

        return gp_object

    def create_element_layers(self, gp_object, element_index):
        # Crear capas de trazo y relleno
        stroke_layer = gp_object.data.layers.new(name=f"{element_index}_stroke", set_active=True)
        fill_layer = gp_object.data.layers.new(name=f"{element_index}_fill", set_active=True)

        # Configurar las sombras y agregar máscaras si es necesario
        for j in range(1, self.num_shadows + 1):
            shadow_layer = gp_object.data.layers.new(name=f"{element_index}_shadow_{j}", set_active=True)
            shadow_layer.blend_mode = 'MULTIPLY'
            shadow_layer.opacity = 0.55
            if self.mask_shadows:
                # Agregar la capa de relleno como máscara a la capa de sombra usando el objeto GPencilLayer
                mask = shadow_layer.mask_layers.add(layer=fill_layer)
                shadow_layer.use_mask_layer = True

    def create_materials(self):
        def create_gpencil_material(name, stroke_color=None, fill_color=None, use_stroke_holdout=False, use_fill_holdout=False):
            # Crear un material específico para Grease Pencil
            if name in bpy.data.materials:
                gp_mat = bpy.data.materials[name]
            else:
                gp_mat = bpy.data.materials.new(name)

            # Crear datos específicos para Grease Pencil si no existen
            if not gp_mat.is_grease_pencil:
                bpy.data.materials.create_gpencil_data(gp_mat)

            # Configurar las propiedades de stroke y fill usando MaterialGPencilStyle
            gp_style = gp_mat.grease_pencil

            if stroke_color:
                gp_style.stroke_style = 'SOLID'
                gp_style.color = stroke_color
                gp_style.show_stroke = True
            else:
                gp_style.show_stroke = False

            if fill_color:
                gp_style.fill_style = 'SOLID'
                gp_style.fill_color = fill_color
                gp_style.show_fill = True
            else:
                gp_style.show_fill = False

            # Configurar el uso de holdout si es necesario
            gp_style.use_stroke_holdout = use_stroke_holdout
            gp_style.use_fill_holdout = use_fill_holdout

        # Crear los materiales de Grease Pencil utilizando el enfoque correcto
        create_gpencil_material("sketch_1", stroke_color=(0, 0.5, 0, 1))
        create_gpencil_material("sketch_2", stroke_color=(0.5, 0, 0, 1))
        create_gpencil_material("line", stroke_color=(0, 0, 0, 1))
        create_gpencil_material("fill", fill_color=(1, 1, 1, 1))
        create_gpencil_material("stroke_shadow", stroke_color=(0.4, 0, 0.8, 1))
        create_gpencil_material("anti_stroke_shadow", stroke_color=(0.4, 0, 0.8, 1), use_stroke_holdout=True)
        create_gpencil_material("fill_shadow", fill_color=(0.4, 0, 0.8, 1))
        create_gpencil_material("anti_fill_shadow", fill_color=(0.4, 0, 0.8, 1), use_fill_holdout=True)


class GPBlankTemplatePanel(bpy.types.Panel):
    bl_label = "Grease Pencil Blank with Template"
    bl_idname = "OBJECT_PT_gpencil_blank_template"
    bl_space_type = 'VIEW_3D'
    bl_region_type = 'UI'
    bl_category = 'Grease Pencil'

    def draw(self, context):
        layout = self.layout
        layout.operator(GPBlankTemplateOperator.bl_idname)


def menu_func(self, context):
    self.layout.operator(GPBlankTemplateOperator.bl_idname, text="Blank with template")


def register():
    bpy.utils.register_class(GPBlankTemplateOperator)
    bpy.utils.register_class(GPBlankTemplatePanel)
    bpy.types.VIEW3D_MT_add.append(menu_func)


def unregister():
    bpy.utils.unregister_class(GPBlankTemplateOperator)
    bpy.utils.unregister_class(GPBlankTemplatePanel)
    bpy.types.VIEW3D_MT_add.remove(menu_func)


if __name__ == "__main__":
    register()
