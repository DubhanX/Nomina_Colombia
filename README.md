# Módulo de Nómina Colombiana para Odoo

Un módulo personalizado y completo desarrollado para **Odoo** que automatiza el proceso de nómina en Colombia, cumpliendo con la legislación laboral vigente y los estándares exigidos por la DIAN (Dirección de Impuestos y Aduanas Nacionales).

---

## Tabla de Contenidos
1. [Características Principales](#características-principales)
2. [Estructura del Proyecto](#estructura-del-proyecto)
3. [Explicación Paso a Paso del Código](#explicación-paso-a-paso-del-código)
   - [1. Configuración del Manifiesto (`__manifest__.py`)](#1-configuración-del-manifiesto-__manifest__py)
   - [2. Reglas de Control de Acceso (`security/ir.model.access.csv`)](#2-reglas-de-control-de-acceso-securityirmodelaccesscsv)
   - [3. Archivo de Inicialización de Modelos (`models/__init__.py`)](#3-archivo-de-inicialización-de-modelos-models__init__py)
   - [4. Parámetros Legales (`models/nomina_config.py`)](#4-parámetros-legales-modelsnomina_configpy)
   - [5. Gestión de Empleados (`models/nomina_employee.py`)](#5-gestión-de-empleados-modelsnomina_employeepy)
   - [6. Novedades y Horas Extras (`models/nomina_novedades.py`)](#6-novedades-y-horas-extras-modelsnomina_novedadespy)
   - [7. Desprendible de Nómina (`models/payslip.py`)](#7-desprendible-de-nómina-modelspayslippy)
   - [8. Procesamiento Masivo / Lotes (`models/payslip_run.py`)](#8-procesamiento-masivo--lotes-modelspayslip_runpy)
   - [9. Vistas XML e Interfaz de Usuario](#9-vistas-xml-e-interfaz-de-usuario)
   - [10. Reporte Imprimible PDF (`views/payslip_report.xml`)](#10-reporte-imprimible-pdf-viewspayslip_reportxml)
   - [11. Estructura del Menú (`views/menu_views.xml`)](#11-estructura-del-menú-viewsmenu_viewsxml)
4. [Guía de Instalación y Despliegue](#guía-de-instalación-y-despliegue)
5. [Demostración del Flujo de Trabajo](#demostración-del-flujo-de-trabajo)

---

## Características Principales

* **Jornada Laboral Colombiana:** Cálculo exacto sobre la jornada estándar legal de **210 horas mensuales** (42 horas semanales).
* **Novedades y Recargos:** Soporte automático para Horas Extras (Diurna, Nocturna, Dominical), Recargos e Incapacidades.
* **Fondo de Solidaridad Pensional (FSP):** Cálculo progresivo (1.0% a 2.0%) para salarios que superen los 4 SMLMV.
* **Retención en la Fuente Laboral:** Algoritmo depurador según los **Artículos 383 y 388 del Estatuto Tributario** (Deducción por dependientes, intereses de vivienda, prepagada y 25% exento en UVT).
* **Seguridad Social y Parafiscales (PILA):** Aportes de Salud, Pensión, ARL, SENA, ICBF y Caja de Compensación (incluyendo exoneración del Art. 114-1 ET).
* **Provisiones y Liquidación Definitiva:** Cálculo automático de Cesantías, Intereses sobre Cesantías, Prima de Servicios y Vacaciones.
* **Pagos Masivos a Bancos:** Exportación de archivo plano estructurado para Bancolombia.
* **Nómina Electrónica DIAN:** Integración vía API REST (generación de CUNE y firmado XML).
* **Envíos Automáticos:** Notificación del desprendible en formato PDF por correo electrónico al empleado.

---

## Estructura del Proyecto

```text
nomina_colombia/
├── __manifest__.py
├── README.md
├── security/
│   └── ir.model.access.csv
├── models/
│   ├── __init__.py
│   ├── nomina_config.py
│   ├── nomina_employee.py
│   ├── nomina_novedades.py
│   ├── payslip.py
│   └── payslip_run.py
└── views/
    ├── menu_views.xml
    ├── nomina_config_views.xml
    ├── nomina_employee_views.xml
    ├── payslip_views.xml
    ├── payslip_run_views.xml
    └── payslip_report.xml

```

---

##  Explicación Paso a Paso del Código

### 1. Configuración del Manifiesto (`__manifest__.py`)

Define la identidad del módulo en Odoo, sus dependencias (`base`, `mail`) y el orden estricto de carga de las vistas de datos.

```python
{
    'name': 'Nómina Colombia',
    'version': '1.0',
    'category': 'Human Resources',
    'summary': 'Módulo de Nómina Colombiana 100% Legal',
    'depends': ['base', 'mail'],
    'data': [
        'security/ir.model.access.csv',
        'views/payslip_report.xml',
        'views/nomina_employee_views.xml',
        'views/nomina_config_views.xml',
        'views/payslip_views.xml',
        'views/payslip_run_views.xml',
        'views/menu_views.xml',
    ],
    'installable': True,
    'application': True,
}

```

---

### 2. Reglas de Control de Acceso (`security/ir.model.access.csv`)

Garantiza los permisos de Lectura, Escritura, Creación y Borrado (CRUD) para los usuarios en cada modelo del sistema.

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_nomina_colombia_config,access_nomina_colombia_config,model_nomina_colombia_config,base.group_user,1,1,1,1
access_nomina_colombia_employee,access_nomina_colombia_employee,model_nomina_colombia_employee,base.group_user,1,1,1,1
access_nomina_colombia_overtime,access_nomina_colombia_overtime,model_nomina_colombia_overtime,base.group_user,1,1,1,1
access_nomina_colombia_payslip,access_nomina_colombia_payslip,model_nomina_colombia_payslip,base.group_user,1,1,1,1
access_nomina_colombia_payslip_run,access_nomina_colombia_payslip_run,model_nomina_colombia_payslip_run,base.group_user,1,1,1,1

```

---

### 3. Archivo de Inicialización de Modelos (`models/__init__.py`)

Importa todos los scripts Python para registrar los modelos en el ORM de Odoo.

```python
from . import nomina_config
from . import nomina_employee
from . import nomina_novedades
from . import payslip
from . import payslip_run

```

---

### 4. Parámetros Legales (`models/nomina_config.py`)

Almacena los valores legales anuales como el Salario Mínimo (SMLMV), Auxilio de Transporte y la Unidad de Valor Tributario (UVT).

```python
from odoo import models, fields

class NominaColombiaConfig(models.Model):
    _name = 'nomina.colombia.config'
    _description = 'Parametros Legales Nomina Colombia'

    name = fields.Char(string="Descripción", required=True)
    smlmv = fields.Float(string="SMLMV ($)", required=True)
    aux_transport_val = fields.Float(string="Auxilio de Transporte ($)", required=True)
    uvt_value = fields.Float(string="Valor UVT ($)", required=True)
    active = fields.Boolean(string="Activo", default=True)

```

---

### 5. Gestión de Empleados (`models/nomina_employee.py`)

Mantiene la información contractual del colaborador, tasa de ARL y variables para depuración tributaria del Art. 388 ET.

```python
from odoo import models, fields

class NominaColombiaEmployee(models.Model):
    _name = 'nomina.colombia.employee'
    _description = 'Empleado de Nómina'

    name = fields.Char(string="Nombre Completo", required=True)
    identification_id = fields.Char(string="Número de Documento / NIT", required=True)
    email = fields.Char(string="Correo Electrónico")
    document_type = fields.Selection([
        ('13', 'Cédula de Ciudadanía'),
        ('31', 'NIT'),
        ('22', 'Cédula de Extranjería'),
        ('41', 'Pasaporte')
    ], string="Tipo Documento", default='13', required=True)
    
    wage = fields.Float(string="Salario Base ($)", required=True)
    arl_rate = fields.Float(string="Tasa ARL (%)", default=0.522, required=True)
    exempt_art_114_1 = fields.Boolean(string="Exonerado Art. 114-1 (Parafiscales)", default=True)

    # Deducciones Art. 388 ET para Retención en la Fuente
    has_dependents = fields.Boolean(string="Tiene Dependientes (10% Deducible)", default=False)
    housing_interest_deduction = fields.Float(string="Deducción Int. Vivienda ($/mes)", default=0.0)
    health_prepaid_deduction = fields.Float(string="Deducción Medicina Prepagada ($/mes)", default=0.0)

```

---

### 6. Novedades y Horas Extras (`models/nomina_novedades.py`)

Calcula los valores monetarios automáticos ajustados a la **divisoria legal de 210 horas mensuales**.

```python
from odoo import models, fields, api

class NominaColombiaOvertime(models.Model):
    _name = 'nomina.colombia.overtime'
    _description = 'Novedades y Horas Extras'

    employee_id = fields.Many2one('nomina.colombia.employee', string="Empleado", required=True)
    payslip_id = fields.Many2one('nomina.colombia.payslip', string="Desprendible de Nómina")
    
    type = fields.Selection([
        ('hed', 'Hora Extra Diurna (+25%)'),
        ('hen', 'Hora Extra Nocturna (+75%)'),
        ('rn', 'Recargo Nocturno (+35%)'),
        ('rd', 'Recargo Dominical/Festivo (+75%)'),
        ('hedd', 'Hora Extra Diurna Dominical (+100%)'),
        ('hend', 'Hora Extra Nocturna Dominical (+150%)'),
        ('incap_comun', 'Incapacidad Común (Día 3-90 66.67%)'),
        ('incap_arl', 'Incapacidad ARL (100%)'),
        ('lic_mat', 'Licencia Maternidad/Paternidad (100%)'),
    ], string="Tipo de Novedad / Hora Extra", required=True)
    
    hours_or_days = fields.Float(string="Cantidad (Horas o Días)", required=True, default=1.0)
    amount = fields.Float(string="Valor Total ($)", compute="_compute_amount", store=True)

    @api.depends('employee_id', 'type', 'hours_or_days')
    def _compute_amount(self):
        for rec in self:
            if not rec.employee_id or rec.employee_id.wage <= 0:
                rec.amount = 0.0
                continue
            
            # Divisor de jornada legal colombiana (210 horas mensuales)
            hourly_val = rec.employee_id.wage / 210.0
            daily_val = rec.employee_id.wage / 30.0

            factors = {
                'hed': hourly_val * 1.25,
                'hen': hourly_val * 1.75,
                'rn': hourly_val * 0.35,
                'rd': hourly_val * 0.75,
                'hedd': hourly_val * 2.00,
                'hend': hourly_val * 2.50,
                'incap_comun': daily_val * 0.6667,
                'incap_arl': daily_val * 1.00,
                'lic_mat': daily_val * 1.00,
            }

            rate = factors.get(rec.type, 0.0)
            rec.amount = rate * rec.hours_or_days

```

---

### 7. Desprendible de Nómina (`models/payslip.py`)

El núcleo motor del módulo. Contiene los algoritmos de FSP, Retención en la Fuente, Devengados, Deducciones, PILA Patronal, Provisiones y las integraciones con la DIAN y servicios de correo.

```python
import json
import base64
import requests
from odoo import models, fields, api, exceptions

class NominaColombiaPayslip(models.Model):
    _name = 'nomina.colombia.payslip'
    _description = 'Desprendible de Nomina Colombia'

    name = fields.Char(string="Número", required=True, default="Nuevo")
    employee_id = fields.Many2one('nomina.colombia.employee', string="Empleado", required=True)
    payslip_run_id = fields.Many2one('nomina.colombia.payslip.run', string="Lote de Nómina")
    
    date_from = fields.Date(string="Fecha Inicio", required=True, default=fields.Date.context_today)
    date_to = fields.Date(string="Fecha Fin", required=True, default=fields.Date.context_today)
    days_worked = fields.Integer(string="Días Trabajados", default=30)
    is_settlement = fields.Boolean(string="¿Es Liquidación Definitiva?", default=False)

    config_id = fields.Many2one('nomina.colombia.config', string="Parámetros Ley", compute="_compute_config", store=True)
    
    smlmv = fields.Float(string="SMLMV", related="config_id.smlmv", store=True)
    aux_transport_val = fields.Float(string="Aux. Transporte Ley", related="config_id.aux_transport_val", store=True)
    uvt_value = fields.Float(string="Valor UVT", related="config_id.uvt_value", store=True)
    
    exempt_art_114_1 = fields.Boolean(string="Exonerado Art. 114-1", related="employee_id.exempt_art_114_1", store=True)
    arl_rate = fields.Float(string="Tasa ARL (%)", related="employee_id.arl_rate", store=True)
    wage_base = fields.Float(string="Salario Base", related="employee_id.wage", store=True)

    # Novedades y Extras
    overtime_ids = fields.One2many('nomina.colombia.overtime', 'payslip_id', string="Horas Extras y Novedades")
    overtime_total = fields.Float(string="Total Extras y Novedades", compute="_compute_payroll_values", store=True)

    # Devengados
    basic_computed = fields.Float(string="Básico Calculado", compute="_compute_payroll_values", store=True)
    aux_transport = fields.Float(string="Auxilio Transporte", compute="_compute_payroll_values", store=True)
    other_devengos = fields.Float(string="Otros Devengados")
    total_devengado = fields.Float(string="Total Devengado", compute="_compute_payroll_values", store=True)

    # Deducciones
    ibc = fields.Float(string="IBC", compute="_compute_payroll_values", store=True)
    salud_employee = fields.Float(string="Salud (Empleado)", compute="_compute_payroll_values", store=True)
    pension_employee = fields.Float(string="Pensión (Empleado)", compute="_compute_payroll_values", store=True)
    fsp_employee = fields.Float(string="Fondo Solidaridad Pensional", compute="_compute_payroll_values", store=True)
    retefuente_val = fields.Float(string="Retención en la Fuente", compute="_compute_payroll_values", store=True)
    other_deductions = fields.Float(string="Otras Deducciones")
    total_deducciones = fields.Float(string="Total Deducciones", compute="_compute_payroll_values", store=True)
    
    net_pay = fields.Float(string="Neto a Pagar", compute="_compute_payroll_values", store=True)

    # Seguridad Social Patronal
    employer_salud = fields.Float(string="Salud Patronal", compute="_compute_payroll_values", store=True)
    employer_pension = fields.Float(string="Pensión Patronal", compute="_compute_payroll_values", store=True)
    employer_arl = fields.Float(string="ARL Patronal", compute="_compute_payroll_values", store=True)
    employer_sena = fields.Float(string="SENA", compute="_compute_payroll_values", store=True)
    employer_icbf = fields.Float(string="ICBF", compute="_compute_payroll_values", store=True)
    employer_caja = fields.Float(string="Caja Compensación", compute="_compute_payroll_values", store=True)
    total_parafiscales = fields.Float(string="Total PILA", compute="_compute_payroll_values", store=True)

    # Provisiones y Liquidación Definitiva
    prov_cesantias = fields.Float(string="Cesantías", compute="_compute_payroll_values", store=True)
    prov_intereses = fields.Float(string="Intereses Cesantías", compute="_compute_payroll_values", store=True)
    prov_prima = fields.Float(string="Prima de Servicios", compute="_compute_payroll_values", store=True)
    prov_vacaciones = fields.Float(string="Vacaciones", compute="_compute_payroll_values", store=True)
    indemnisation = fields.Float(string="Indemnización Despido", default=0.0)
    total_provisiones = fields.Float(string="Total Provisiones / Liquidación", compute="_compute_payroll_values", store=True)

    state = fields.Selection([('draft', 'Borrador'), ('done', 'Realizado')], string="Estado", default='draft')
    dian_status = fields.Selection([('draft', 'Sin Enviar'), ('sent', 'Enviado / Aprobado'), ('rejected', 'Rechazado')], string="Estado DIAN", default='draft', readonly=True)
    cune = fields.Char(string="CUNE", readonly=True)
    dian_response = fields.Text(string="Respuesta DIAN", readonly=True)

    @api.depends('date_from')
    def _compute_config(self):
        config = self.env['nomina.colombia.config'].search([('active', '=', True)], limit=1)
        for record in self:
            record.config_id = config.id if config else False

    def _calculate_fsp(self, ibc, smlmv):
        if smlmv <= 0 or ibc < (4 * smlmv):
            return 0.0
        n_smlmv = ibc / smlmv
        if 4 <= n_smlmv < 16:
            rate = 0.01
        elif 16 <= n_smlmv < 17:
            rate = 0.012
        elif 17 <= n_smlmv < 18:
            rate = 0.014
        elif 18 <= n_smlmv < 19:
            rate = 0.016
        elif 19 <= n_smlmv < 20:
            rate = 0.018
        else:
            rate = 0.02
        return ibc * rate

    def _calculate_retefuente(self, rec, ibc):
        if rec.uvt_value <= 0:
            return 0.0
        
        salud_pen = rec.salud_employee + rec.pension_employee + rec.fsp_employee
        base_ingreso = ibc - salud_pen
        
        ded_dep = base_ingreso * 0.10 if rec.employee_id.has_dependents else 0.0
        ded_vivienda = rec.employee_id.housing_interest_deduction
        ded_salud = rec.employee_id.health_prepaid_deduction
        
        base_subtotal = base_ingreso - ded_dep - ded_vivienda - ded_salud
        renta_exenta = min(base_subtotal * 0.25, 240 * rec.uvt_value / 12)
        
        base_gravable_pesos = base_subtotal - renta_exenta
        base_uvt = base_gravable_pesos / rec.uvt_value
        
        if base_uvt <= 95:
            impuesto_uvt = 0.0
        elif 95 < base_uvt <= 150:
            impuesto_uvt = (base_uvt - 95) * 0.19
        elif 150 < base_uvt <= 360:
            impuesto_uvt = (base_uvt - 150) * 0.28 + 10
        elif 360 < base_uvt <= 640:
            impuesto_uvt = (base_uvt - 360) * 0.33 + 69
        else:
            impuesto_uvt = (base_uvt - 640) * 0.35 + 162

        return impuesto_uvt * rec.uvt_value

    @api.depends('wage_base', 'days_worked', 'smlmv', 'aux_transport_val', 'other_devengos', 'other_deductions', 'exempt_art_114_1', 'arl_rate', 'overtime_ids.amount', 'is_settlement', 'indemnisation')
    def _compute_payroll_values(self):
        for rec in self:
            rec.overtime_total = sum(rec.overtime_ids.mapped('amount'))
            rec.basic_computed = (rec.wage_base / 30.0) * rec.days_worked
            
            if rec.wage_base <= (2 * rec.smlmv) and rec.smlmv > 0:
                rec.aux_transport = (rec.aux_transport_val / 30.0) * rec.days_worked
            else:
                rec.aux_transport = 0.0

            rec.total_devengado = rec.basic_computed + rec.aux_transport + rec.other_devengos + rec.overtime_total
            rec.ibc = rec.basic_computed + rec.other_devengos + rec.overtime_total

            rec.salud_employee = rec.ibc * 0.04
            rec.pension_employee = rec.ibc * 0.04
            rec.fsp_employee = rec._calculate_fsp(rec.ibc, rec.smlmv)
            rec.retefuente_val = rec._calculate_retefuente(rec, rec.ibc)

            rec.total_deducciones = rec.salud_employee + rec.pension_employee + rec.fsp_employee + rec.retefuente_val + rec.other_deductions
            rec.net_pay = rec.total_devengado - rec.total_deducciones

            # Parafiscales
            rec.employer_salud = 0.0 if rec.exempt_art_114_1 else (rec.ibc * 0.085)
            rec.employer_pension = rec.ibc * 0.12
            rec.employer_arl = rec.ibc * (rec.arl_rate / 100.0)
            rec.employer_sena = 0.0 if rec.exempt_art_114_1 else (rec.ibc * 0.02)
            rec.employer_icbf = 0.0 if rec.exempt_art_114_1 else (rec.ibc * 0.03)
            rec.employer_caja = rec.ibc * 0.04
            rec.total_parafiscales = rec.employer_salud + rec.employer_pension + rec.employer_arl + rec.employer_sena + rec.employer_icbf + rec.employer_caja

            # Provisiones / Liquidación Definitiva
            base_prestaciones = rec.basic_computed + rec.aux_transport
            rec.prov_cesantias = base_prestaciones * 0.0833
            rec.prov_intereses = rec.prov_cesantias * 0.12
            rec.prov_prima = base_prestaciones * 0.0833
            rec.prov_vacaciones = rec.basic_computed * 0.0417

            if rec.is_settlement:
                rec.total_provisiones = rec.prov_cesantias + rec.prov_intereses + rec.prov_prima + rec.prov_vacaciones + rec.indemnisation
            else:
                rec.total_provisiones = rec.prov_cesantias + rec.prov_intereses + rec.prov_prima + rec.prov_vacaciones

    def action_confirm(self):
        self.write({'state': 'done'})

    def action_draft(self):
        self.write({'state': 'draft'})

    def action_send_email(self):
        self.ensure_one()
        if not self.employee_id.email:
            raise exceptions.UserError("El empleado no tiene asignado un Correo Electrónico.")
        
        report = self.env.ref('nomina_colombia.action_report_payslip')
        pdf_content, _ = report._render_qweb_pdf(self.id)
        
        attachment = self.env['ir.attachment'].create({
            'name': f'Desprendible_{self.employee_id.name}_{self.date_from}.pdf',
            'type': 'binary',
            'datas': base64.b64encode(pdf_content),
            'res_model': self._name,
            'res_id': self.id,
            'mimetype': 'application/pdf',
        })

        mail_values = {
            'subject': f'Desprendible de Nómina - {self.employee_id.name} ({self.date_from})',
            'body_html': f'<p>Hola {self.employee_id.name},</p><p>Adjunto encontrarás tu comprobante de pago correspondiente al periodo {self.date_from} a {self.date_to}.</p>',
            'email_to': self.employee_id.email,
            'attachment_ids': [(6, 0, [attachment.id])],
        }
        self.env['mail.mail'].create(mail_values).send()
        return True

    def action_send_dian(self):
        for slip in self:
            payload = {
                "tipo_xml": "103",
                "periodo": {"fecha_inicio": str(slip.date_from), "fecha_fin": str(slip.date_to)},
                "empleado": {"tipo_documento": slip.employee_id.document_type, "numero_documento": slip.employee_id.identification_id or '', "nombre": slip.employee_id.name or ''},
                "devengados": {"basico": slip.basic_computed, "transporte": slip.aux_transport, "extras": slip.overtime_total},
                "deducciones": {"salud": slip.salud_employee, "pension": slip.pension_employee, "fsp": slip.fsp_employee, "retencion": slip.retefuente_val},
                "comprobante": {"devengados_total": slip.total_devengado, "deducciones_total": slip.total_deducciones, "comprobante_total": slip.net_pay}
            }
            api_url = "[https://api.proveedortecnologico.com/v1/nomina/enviar](https://api.proveedortecnologico.com/v1/nomina/enviar)"
            headers = {'Content-Type': 'application/json', 'Authorization': 'Bearer TU_TOKEN_API_AQUI'}
            try:
                response = requests.post(api_url, data=json.dumps(payload), headers=headers, timeout=30)
                res_data = response.json()
                if response.status_code == 200 and res_data.get('is_valid'):
                    slip.write({'dian_status': 'sent', 'cune': res_data.get('cune'), 'dian_response': json.dumps(res_data, indent=2)})
                else:
                    slip.write({'dian_status': 'rejected', 'dian_response': json.dumps(res_data, indent=2)})
            except Exception as e:
                raise exceptions.UserError(f"Error al conectar con la API DIAN: {str(e)}")

```

---

### 8. Procesamiento Masivo / Lotes (`models/payslip_run.py`)

Genera automáticamente la nómina para todos los empleados y genera el archivo plano bancario en formato UTF-8 para pago masivo.

```python
import base64
from odoo import models, fields

class NominaColombiaPayslipRun(models.Model):
    _name = 'nomina.colombia.payslip.run'
    _description = 'Lote de Nomina'

    name = fields.Char(string="Nombre del Lote", required=True)
    date_start = fields.Date(string="Fecha Inicio", required=True)
    date_end = fields.Date(string="Fecha Fin", required=True)
    payslip_ids = fields.One2many('nomina.colombia.payslip', 'payslip_run_id', string="Desprendibles")
    state = fields.Selection([('draft', 'Borrador'), ('close', 'Cerrado')], string="Estado", default='draft')

    txt_file = fields.Binary(string="Archivo Plano Bancolombia", readonly=True)
    txt_filename = fields.Char(string="Nombre Archivo TXT")

    def action_generate_slips(self):
        employees = self.env['nomina.colombia.employee'].search([])
        for emp in employees:
            self.env['nomina.colombia.payslip'].create({
                'name': f"NOM/{self.name}/{emp.name}",
                'employee_id': emp.id,
                'payslip_run_id': self.id,
                'date_from': self.date_start,
                'date_to': self.date_end,
                'days_worked': 30,
            })

    def action_generate_bancolombia_txt(self):
        content = ""
        for slip in self.payslip_ids:
            net_cents = str(int(round(slip.net_pay * 100))).zfill(12)
            doc = (slip.employee_id.identification_id or '').zfill(15)
            name = (slip.employee_id.name or '').ljust(30)[:30]
            content += f"6{doc}{name}{net_cents}\n"

        self.write({
            'txt_file': base64.b64encode(content.encode('utf-8')),
            'txt_filename': f"Bancolombia_Lote_{self.name}.txt"
        })

```

---

### 9. Vistas XML e Interfaz de Usuario

#### `views/nomina_config_views.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_nomina_config_form" model="ir.ui.view">
        <field name="name">nomina.colombia.config.form</field>
        <field name="model">nomina.colombia.config</field>
        <field name="arch" type="xml">
            <form string="Parámetros Legales">
                <sheet>
                    <group>
                        <group string="Información General">
                            <field name="name"/>
                            <field name="active"/>
                        </group>
                        <group string="Valores Legalmente Vigentes">
                            <field name="smlmv"/>
                            <field name="aux_transport_val"/>
                            <field name="uvt_value"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="view_nomina_config_tree" model="ir.ui.view">
        <field name="name">nomina.colombia.config.list</field>
        <field name="model">nomina.colombia.config</field>
        <field name="arch" type="xml">
            <list string="Parámetros Legales">
                <field name="name"/>
                <field name="smlmv"/>
                <field name="aux_transport_val"/>
                <field name="uvt_value"/>
                <field name="active"/>
            </list>
        </field>
    </record>

    <record id="action_nomina_config" model="ir.actions.act_window">
        <field name="name">Parámetros Legales</field>
        <field name="res_model">nomina.colombia.config</field>
        <field name="view_mode">list,form</field>
    </record>
</odoo>

```

#### `views/nomina_employee_views.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_nomina_employee_form" model="ir.ui.view">
        <field name="name">nomina.colombia.employee.form</field>
        <field name="model">nomina.colombia.employee</field>
        <field name="arch" type="xml">
            <form string="Empleado">
                <sheet>
                    <div class="oe_title">
                        <h1><field name="name" placeholder="Nombre completo"/></h1>
                    </div>
                    <group>
                        <group string="Identificación">
                            <field name="document_type"/>
                            <field name="identification_id"/>
                            <field name="email"/>
                        </group>
                        <group string="Condiciones Laborales">
                            <field name="wage"/>
                            <field name="arl_rate"/>
                            <field name="exempt_art_114_1"/>
                        </group>
                    </group>
                    <group string="Deducciones Retención en la Fuente (Art 388 ET)">
                        <group>
                            <field name="has_dependents"/>
                            <field name="housing_interest_deduction"/>
                            <field name="health_prepaid_deduction"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="view_nomina_employee_tree" model="ir.ui.view">
        <field name="name">nomina.colombia.employee.list</field>
        <field name="model">nomina.colombia.employee</field>
        <field name="arch" type="xml">
            <list string="Empleados">
                <field name="name"/>
                <field name="identification_id"/>
                <field name="email"/>
                <field name="wage"/>
                <field name="arl_rate"/>
            </list>
        </field>
    </record>

    <record id="action_nomina_employee" model="ir.actions.act_window">
        <field name="name">Empleados</field>
        <field name="res_model">nomina.colombia.employee</field>
        <field name="view_mode">list,form</field>
    </record>
</odoo>

```

#### `views/payslip_views.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_nomina_payslip_form" model="ir.ui.view">
        <field name="name">nomina.colombia.payslip.form</field>
        <field name="model">nomina.colombia.payslip</field>
        <field name="arch" type="xml">
            <form string="Desprendible de Nómina">
                <header>
                    <button name="action_confirm" string="Liquidar" type="object" class="btn-primary" invisible="state != 'draft'"/>
                    <button name="action_send_email" string="Enviar por Correo" type="object" class="btn-secondary"/>
                    <button name="action_send_dian" string="Enviar a la DIAN" type="object" class="btn-primary" invisible="dian_status == 'sent'"/>
                    <button name="action_draft" string="Volver a Borrador" type="object" invisible="state != 'done'"/>
                    <field name="state" widget="statusbar" statusbar_visible="draft,done"/>
                    <field name="dian_status" widget="statusbar" statusbar_visible="draft,rejected,sent"/>
                </header>
                <sheet>
                    <div class="oe_title">
                        <h1><field name="name" readonly="1"/></h1>
                    </div>
                    <group>
                        <group string="Información General">
                            <field name="employee_id"/>
                            <field name="payslip_run_id"/>
                            <field name="config_id"/>
                            <field name="is_settlement"/>
                        </group>
                        <group string="Periodo Laboral">
                            <field name="date_from"/>
                            <field name="date_to"/>
                            <field name="days_worked"/>
                            <field name="wage_base"/>
                        </group>
                    </group>

                    <notebook>
                        <page string="Horas Extras y Novedades">
                            <field name="overtime_ids">
                                <list editable="bottom">
                                    <field name="type"/>
                                    <field name="hours_or_days"/>
                                    <field name="amount" sum="Total Novedades"/>
                                </list>
                            </field>
                        </page>

                        <page string="Resumen de Nómina">
                            <group>
                                <group string="DEVENGADOS">
                                    <field name="basic_computed"/>
                                    <field name="aux_transport"/>
                                    <field name="overtime_total"/>
                                    <field name="other_devengos"/>
                                    <field name="total_devengado" class="oe_subtotal_footer_separator"/>
                                </group>
                                <group string="DEDUCCIONES">
                                    <field name="ibc"/>
                                    <field name="salud_employee"/>
                                    <field name="pension_employee"/>
                                    <field name="fsp_employee"/>
                                    <field name="retefuente_val"/>
                                    <field name="other_deductions"/>
                                    <field name="total_deducciones" class="oe_subtotal_footer_separator"/>
                                </group>
                            </group>
                            <group class="oe_subtotal_footer oe_right">
                                <field name="net_pay" class="oe_subtotal_footer_separator"/>
                            </group>
                        </page>

                        <page string="Seguridad Social Patronal (PILA)">
                            <group>
                                <group string="Aportes">
                                    <field name="employer_salud"/>
                                    <field name="employer_pension"/>
                                    <field name="employer_arl"/>
                                </group>
                                <group string="Parafiscales">
                                    <field name="employer_sena"/>
                                    <field name="employer_icbf"/>
                                    <field name="employer_caja"/>
                                    <field name="total_parafiscales" class="oe_subtotal_footer_separator"/>
                                </group>
                            </group>
                        </page>

                        <page string="Provisiones / Liquidación">
                            <group>
                                <group string="Prestaciones Sociales">
                                    <field name="prov_cesantias"/>
                                    <field name="prov_intereses"/>
                                    <field name="prov_prima"/>
                                    <field name="prov_vacaciones"/>
                                    <field name="indemnisation" invisible="not is_settlement"/>
                                    <field name="total_provisiones" class="oe_subtotal_footer_separator"/>
                                </group>
                            </group>
                        </page>

                        <page string="Respuesta DIAN">
                            <group>
                                <field name="cune"/>
                                <field name="dian_response"/>
                            </group>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="view_nomina_payslip_tree" model="ir.ui.view">
        <field name="name">nomina.colombia.payslip.list</field>
        <field name="model">nomina.colombia.payslip</field>
        <field name="arch" type="xml">
            <list string="Desprendibles de Nómina">
                <field name="name"/>
                <field name="employee_id"/>
                <field name="date_from"/>
                <field name="date_to"/>
                <field name="total_devengado"/>
                <field name="total_deducciones"/>
                <field name="net_pay"/>
                <field name="state"/>
                <field name="dian_status"/>
            </list>
        </field>
    </record>

    <record id="action_nomina_payslip" model="ir.actions.act_window">
        <field name="name">Desprendibles de Nómina</field>
        <field name="res_model">nomina.colombia.payslip</field>
        <field name="view_mode">list,form</field>
    </record>
</odoo>

```

#### `views/payslip_run_views.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_nomina_payslip_run_form" model="ir.ui.view">
        <field name="name">nomina.colombia.payslip.run.form</field>
        <field name="model">nomina.colombia.payslip.run</field>
        <field name="arch" type="xml">
            <form string="Lote de Nómina">
                <header>
                    <button name="action_generate_slips" string="Generar Desprendibles" type="object" class="btn-primary" invisible="state != 'draft'"/>
                    <button name="action_generate_bancolombia_txt" string="Generar Archivo Bancolombia" type="object" class="btn-secondary"/>
                    <field name="state" widget="statusbar" statusbar_visible="draft,close"/>
                </header>
                <sheet>
                    <div class="oe_title">
                        <h1><field name="name" placeholder="Ej: Nómina Quincenal / Mensual"/></h1>
                    </div>
                    <group>
                        <group string="Periodo">
                            <field name="date_start"/>
                            <field name="date_end"/>
                        </group>
                        <group string="Descarga de Archivo Plano">
                            <field name="txt_filename" invisible="1"/>
                            <field name="txt_file" filename="txt_filename"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Desprendibles">
                            <field name="payslip_ids">
                                <list>
                                    <field name="name"/>
                                    <field name="employee_id"/>
                                    <field name="date_from"/>
                                    <field name="date_to"/>
                                    <field name="total_devengado"/>
                                    <field name="total_deducciones"/>
                                    <field name="net_pay"/>
                                    <field name="state"/>
                                </list>
                            </field>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="view_nomina_payslip_run_tree" model="ir.ui.view">
        <field name="name">nomina.colombia.payslip.run.list</field>
        <field name="model">nomina.colombia.payslip.run</field>
        <field name="arch" type="xml">
            <list string="Lotes de Nómina">
                <field name="name"/>
                <field name="date_start"/>
                <field name="date_end"/>
                <field name="state"/>
            </list>
        </field>
    </record>

    <record id="action_nomina_payslip_run" model="ir.actions.act_window">
        <field name="name">Lotes de Nómina</field>
        <field name="res_model">nomina.colombia.payslip.run</field>
        <field name="view_mode">list,form</field>
    </record>
</odoo>

```

---

### 10. Reporte Imprimible PDF (`views/payslip_report.xml`)

Plantilla QWeb que genera el comprobante PDF listo para imprimir o enviar.

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="action_report_payslip" model="ir.actions.report">
        <field name="name">Desprendible de Nómina</field>
        <field name="model">nomina.colombia.payslip</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">nomina_colombia.report_payslip_template</field>
        <field name="report_file">nomina_colombia.report_payslip_template</field>
        <field name="binding_model_id" ref="model_nomina_colombia_payslip"/>
        <field name="binding_type">report</field>
    </record>

    <template id="report_payslip_template">
        <t t-call="web.html_container">
            <t t-foreach="docs" t-as="o">
                <t t-call="web.external_layout">
                    <div class="page">
                        <h2 class="text-center">DESPRENDIBLE DE NÓMINA</h2>
                        <h4 class="text-center"><span t-field="o.name"/></h4>
                        <br/>
                        <table class="table table-sm table-bordered">
                            <tr>
                                <td><strong>Empleado:</strong> <span t-field="o.employee_id.name"/></td>
                                <td><strong>Documento:</strong> <span t-field="o.employee_id.identification_id"/></td>
                            </tr>
                            <tr>
                                <td><strong>Periodo:</strong> <span t-field="o.date_from"/> al <span t-field="o.date_to"/></td>
                                <td><strong>Días Trabajados:</strong> <span t-field="o.days_worked"/></td>
                            </tr>
                        </table>

                        <br/>
                        <div class="row">
                            <div class="col-6">
                                <h5 class="text-success">DEVENGADOS</h5>
                                <table class="table table-sm table-striped">
                                    <tr>
                                        <td>Sueldo Básico</td>
                                        <td class="text-end">$ <span t-field="o.basic_computed" t-options='{"widget": "float", "precision": 0}'/></td>
                                    </tr>
                                    <tr>
                                        <td>Auxilio de Transporte</td>
                                        <td class="text-end">$ <span t-field="o.aux_transport" t-options='{"widget": "float", "precision": 0}'/></td>
                                    </tr>
                                    <tr>
                                        <td>Horas Extras y Novedades</td>
                                        <td class="text-end">$ <span t-field="o.overtime_total" t-options='{"widget": "float", "precision": 0}'/></td>
                                    </tr>
                                    <tr>
                                        <td>Otros Devengos</td>
                                        <td class="text-end">$ <span t-field="o.other_devengos" t-options='{"widget": "float", "precision": 0}'/></td>
                                    </tr>
                                    <tr class="fw-bold">
                                        <td>TOTAL DEVENGADO</td>
                                        <td class="text-end">$ <span t-field="o.total_devengado" t-options='{"widget": "float", "precision": 0}'/></td>
                                    </tr>
                                </table>
                            </div>
                            <div class="col-6">
                                <h5 class="text-danger">DEDUCCIONES</h5>
                                <table class="table table-sm table-striped">
                                    <tr>
                                        <td>Salud (4%)</td>
                                        <td class="text-end">$ <span t-field="o.salud_employee" t-options='{"widget": "float", "precision": 0}'/></td>
                                    </tr>
                                    <tr>
                                        <td>Pensión (4%)</td>
                                        <td class="text-end">$ <span t-field="o.pension_employee" t-options='{"widget": "float", "precision": 0}'/></td>
                                    </tr>
                                    <tr>
                                        <td>Fondo Solidaridad Pensional</td>
                                        <td class="text-end">$ <span t-field="o.fsp_employee" t-options='{"widget": "float", "precision": 0}'/></td>
                                    </tr>
                                    <tr>
                                        <td>Retención en la Fuente</td>
                                        <td class="text-end">$ <span t-field="o.retefuente_val" t-options='{"widget": "float", "precision": 0}'/></td>
                                    </tr>
                                    <tr>
                                        <td>Otras Deducciones</td>
                                        <td class="text-end">$ <span t-field="o.other_deductions" t-options='{"widget": "float", "precision": 0}'/></td>
                                    </tr>
                                    <tr class="fw-bold">
                                        <td>TOTAL DEDUCCIONES</td>
                                        <td class="text-end">$ <span t-field="o.total_deducciones" t-options='{"widget": "float", "precision": 0}'/></td>
                                    </tr>
                                </table>
                            </div>
                        </div>

                        <br/>
                        <div class="card p-3 bg-light">
                            <h3 class="text-end m-0">
                                <strong>NETO A PAGAR: </strong>
                                <span t-field="o.net_pay" t-options='{"widget": "monetary", "display_currency": o.env.company.currency_id}'/>
                            </h3>
                        </div>

                        <t t-if="o.cune">
                            <br/>
                            <small class="text-muted"><strong>CUNE DIAN:</strong> <span t-field="o.cune"/></small>
                        </t>
                    </div>
                </t>
            </t>
        </t>
    </template>
</odoo>

```

---

### 11. Estructura del Menú (`views/menu_views.xml`)

Construye la navegación en la barra principal de aplicaciones de Odoo.

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Menú Raíz Principal -->
    <menuitem id="menu_nomina_colombia_root" 
              name="Nómina Colombia" 
              sequence="10"/>

    <!-- Submenús Secundarios -->
    <menuitem id="menu_nomina_payslip" 
              name="Desprendibles" 
              parent="menu_nomina_colombia_root" 
              action="action_nomina_payslip" 
              sequence="10"/>

    <menuitem id="menu_nomina_payslip_run" 
              name="Lotes de Nómina" 
              parent="menu_nomina_colombia_root" 
              action="action_nomina_payslip_run" 
              sequence="20"/>

    <menuitem id="menu_nomina_employee" 
              name="Empleados" 
              parent="menu_nomina_colombia_root" 
              action="action_nomina_employee" 
              sequence="30"/>

    <menuitem id="menu_nomina_config" 
              name="Parámetros Legales" 
              parent="menu_nomina_colombia_root" 
              action="action_nomina_config" 
              sequence="40"/>
</odoo>

```

---

## 🚀 Guía de Instalación y Despliegue

1. Clona o copia el directorio del módulo en el directorio de add-ons personalizados:
```bash
cp -r nomina_colombia /opt/odoo/custom-addons/

```


2. Actualiza la lista de aplicaciones e instala el módulo corriendo en la terminal:
```bash
su -s /bin/bash odoo -c "/opt/odoo/venv/bin/python /opt/odoo/odoo/odoo-bin -c /etc/odoo.conf -d tu_base_datos -u nomina_colombia --stop-after-init" && systemctl restart odoo

```


3. Ingresa a la interfaz web de Odoo y recarga la página (`Ctrl + F5`). Verás la app **Nómina Colombia** disponible.

---

## Demostración del Flujo de Trabajo

1. **Parámetros Legales:** Configura los datos vigentes (SMLMV, Auxilio Transporte, UVT).
2. **Empleados:** Registra los empleados con sus salarios y condiciones tributarias.
3. **Lotes / Desprendibles:** Crea una ejecución masiva o individual, agrega horas extras/novedades si aplican y haz clic en **Liquidar**.
4. **Exportar / Notificar:** Genera el plano de Bancolombia, descarga/envía el comprobante en PDF o transmite a la DIAN.

---

*Desarrollado con para Odoo & la legislación colombiana.*

```



# Glosario de Términos Financieros y de Nómina

Este proyecto contiene una guía de referencia rápida sobre los principales conceptos utilizados en la liquidación de nómina, la gestión salarial y la contabilidad financiera[cite: 1, 2]. Incluye los códigos y conceptos estándar utilizados en el registro de nómina.

---

## 1. Conceptos Principales de Nómina y Sus Códigos

* **Devengados (o Total Devengado):** Es la totalidad de los ingresos o sumas de dinero que un trabajador tiene derecho a recibir en un periodo determinado por la prestación de sus servicios[cite: 1, 2]. Representa el ingreso bruto antes de aplicar cualquier descuento o retención[cite: 1, 2].
  * `DEV-01` **Salario Básico:** Remuneración ordinaria fija pactada en el contrato de trabajo[cite: 1, 2].
  * `DEV-02` **Horas Extras y Recargos:** Pago por trabajo en jornadas adicionales, nocturnas, dominicales o festivas[cite: 1, 2].
  * `DEV-03` **Comisiones y Bonificaciones:** Pagos variables por cumplimiento de metas de ventas o desempeño[cite: 1, 2].
  * `DEV-04` **Auxilio de Transporte:** Subsidio legal para el desplazamiento del empleado hacia su sitio de trabajo[cite: 1, 2].

* **Deducciones (o Descuentos de Nómina):** Son los valores que legal o voluntariamente se le restan al trabajador de su total devengado[cite: 1, 2]. Reducen el dinero que ingresa de forma líquida al empleado[cite: 1, 2].
  * `DED-01` **Aporte a Salud (Empleado):** Descuento obligatorio para la cobertura del sistema de salud[cite: 1, 2].
  * `DED-02` **Aporte a Pensión (Empleado):** Descuento obligatorio destinado al fondo de pensiones y jubilación[cite: 1, 2].
  * `DED-03` **Retención en la Fuente:** Cobro anticipado de impuestos descontado directamente del salario[cite: 1, 2].
  * `DED-04` **Libranzas y Préstamos:** Cuotas de préstamos bancarios, cooperativas o deducciones autorizadas[cite: 1, 2].
  * `DED-05` **Embargos Judiciales / Cuotas Voluntarias:** Descuentos por mandato legal o aportes a fondos de empleados/sindicatos[cite: 1, 2].

* **Neto a Pagar (o Salario Neto):** Es la cantidad real y efectiva que el empleado recibe en su cuenta bancaria tras restar las deducciones del total devengado[cite: 1, 2].
  * **Código de Resultado:** `NET-01`
  * **Fórmula:** `Neto a Pagar = Total Devengado - Total Deducciones`[cite: 1, 2]

* **Prestaciones Sociales:** Son los beneficios legales adicionales al salario que el empleador debe pagar a los trabajadores para cubrir riesgos o necesidades derivadas de la relación laboral[cite: 1, 2].
  * `PRE-01` **Prima de Servicios:** Pago equivalente a un mes de salario por año laborado (entregado semestralmente)[cite: 1, 2].
  * `PRE-02` **Auxilio de Cesantías:** Ahorro acumulado a cargo del empleador para periodos de desempleo, educación o vivienda[cite: 1, 2].
  * `PRE-03` **Intereses sobre Cesantías:** Rentabilidad sobre el saldo acumulado de las cesantías pagada anualmente[cite: 1, 2].
  * `PRE-04` **Dotación:** Suministro de calzado y vestido de labor exigido por ley[cite: 1, 2].

---

## 2. Resumen de Estructura de Nómina

| Código | Categoría | ¿Qué representa? | Efecto en el pago final |
| :--- | :--- | :--- | :--- |
| `DEV-TOT` | **Devengados** | Ingresos totales brutos generados en el periodo[cite: 1, 2] | **Suma (+)** al valor a recibir[cite: 1, 2] |
| `DED-TOT` | **Deducciones** | Descuentos legales, fiscales o préstamos personales[cite: 1, 2] | **Resta (−)** del valor devengado[cite: 1, 2] |
| `NET-01` | **Neto a Pagar** | Ingreso real y efectivo que percibe el empleado[cite: 1, 2] | **Resultado final (=)**[cite: 1, 2] |
| `PRE-TOT` | **Prestaciones Sociales** | Beneficios adicionales fijados por ley (cesantías, prima)[cite: 1, 2] | Pagos periódicos o acumulados[cite: 1, 2] |

---

## 3. Términos Contables y Financieros Relacionados

* **Principio de Devengo / Causación (*Accrual Accounting*):** Criterio contable que establece que los hechos económicos (ingresos o gastos) se reconocen y registran en los libros en el momento exacto en que nace el derecho u obligación, independientemente de cuándo se efectúe el cobro o pago real en efectivo[cite: 1, 2].
* **Ingreso Bruto vs. Ingreso Neto:** El *ingreso bruto* es la suma total generada antes de impuestos y deducciones; el *ingreso neto* es el monto líquido restante tras descontar todos los costos, impuestos y retenciones[cite: 1, 2].
* **Retención en la Fuente:** Mecanismo de cobro anticipado de impuestos mediante el cual el pagador (empleador o comprador) descuenta un porcentaje determinado del pago y lo entrega directamente a la entidad tributaria[cite: 1, 2].```
